---
layout: post
title: "Écrire une règle de détection Wazuh et la tester"
description: "Créer une règle de corrélation pour détecter des échecs SSH répétés, la valider avec wazuh-logtest, puis la déployer."
categories: [wazuh, détection]
---

Le jeu de règles livré avec Wazuh couvre beaucoup de cas, mais on a vite besoin d'une règle à soi : un seuil différent, une application maison, un comportement propre à l'environnement. Voici la méthode, avec un exemple simple : plusieurs échecs d'authentification SSH depuis la même adresse.

Wazuh contient déjà une règle proche pour le brute force SSH (5763 dans le jeu par défaut). L'intérêt ici est d'apprendre le mécanisme, puis de l'appliquer à des cas que le jeu par défaut ne couvre pas.

## Où écrire ses règles

Les règles personnalisées vont dans `/var/ossec/etc/rules/local_rules.xml`. Les identifiants de **100000 à 120000** sont réservés à cet usage, ce qui évite tout conflit avec les règles officielles.

## Étape 1 : observer la règle de base

Une règle de corrélation s'appuie sur une règle qui reconnaît déjà l'événement individuel. Lancez l'outil de test et collez une ligne de journal SSH :

```bash
sudo /var/ossec/bin/wazuh-logtest
```

```text
Oct  9 10:12:01 srv sshd[1234]: Failed password for root from 203.0.113.10 port 52144 ssh2
```

L'outil affiche les trois phases de l'analyse : pré-décodage, décodage, puis la règle déclenchée avec son identifiant. Pour un utilisateur existant, il s'agit généralement de la règle **5760** (« sshd: authentication failed »). Notez l'identifiant **que vous voyez réellement** : c'est lui qui sert de base à la suite.

## Étape 2 : écrire la règle de corrélation

```xml
<group name="local,sshd,">
  <rule id="100100" level="10" frequency="5" timeframe="60">
    <if_matched_sid>5760</if_matched_sid>
    <same_source_ip />
    <description>sshd: 5 échecs d'authentification en 60 s depuis la même IP</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
```

Ce que fait chaque élément :

- `frequency="5"` et `timeframe="60"` : la règle se déclenche à la cinquième occurrence dans une fenêtre de 60 secondes
- `if_matched_sid` : l'événement à compter (ici la règle de base notée à l'étape 1)
- `same_source_ip` : les occurrences doivent venir de la même adresse, sinon des échecs légitimes d'utilisateurs différents s'additionneraient
- `level` : la gravité, de 0 à 15. À partir de 10, on est sur des alertes qu'on veut voir remonter
- `mitre` : rattache l'alerte à la technique ATT&CK T1110 (force brute), utile pour filtrer et présenter les résultats

## Étape 3 : valider avant de redémarrer

Vérifiez d'abord que la configuration est valide :

```bash
sudo /var/ossec/bin/wazuh-analysisd -t
```

Puis redémarrez le gestionnaire :

```bash
sudo systemctl restart wazuh-manager
```

## Étape 4 : déclencher la règle

Dans `wazuh-logtest`, collez la même ligne de journal plusieurs fois de suite en quelques secondes. À la cinquième, la règle 100100 doit s'afficher. Ensuite, sur une machine de votre lab, tentez cinq connexions SSH avec un mauvais mot de passe et vérifiez l'alerte dans le tableau de bord.

## Les pièges classiques

- **Aucune alerte** : l'identifiant dans `if_matched_sid` ne correspond pas à la règle réellement déclenchée. Reprenez l'étape 1
- **Erreur au redémarrage** : une balise XML mal fermée. Le test avec `wazuh-analysisd -t` l'indique avant que le service tombe
- **Trop d'alertes** : le seuil est trop bas pour votre environnement. Montez `frequency` ou allongez `timeframe`

Une règle n'est terminée que lorsqu'elle est testée dans les deux sens : elle se déclenche quand elle le doit, et reste silencieuse sinon.
