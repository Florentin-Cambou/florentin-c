---
layout: post
title: "Wazuh : réduire le bruit pour garder les alertes utiles"
description: "Cinq habitudes pour qu'un tableau de bord Wazuh reste lisible : silencer proprement, surveiller peu mais bien, tester chaque changement."
categories: [wazuh, conseils]
---

Un Wazuh fraîchement installé produit beaucoup d'alertes. Si tout est important, plus rien ne l'est : l'analyste cesse de regarder. Voici cinq habitudes pour garder un signal exploitable.

## 1. Silencer une règle sans la supprimer

Ne modifiez pas les fichiers de règles officiels : ils sont écrasés à chaque mise à jour. Écrivez plutôt une règle de niveau 0 dans `local_rules.xml`. Un niveau 0 ne génère aucune alerte.

```xml
<rule id="100200" level="0">
  <if_sid>5402</if_sid>
  <field name="srcuser">compte_de_sauvegarde</field>
  <description>sudo attendu : compte de sauvegarde</description>
</rule>
```

Ce modèle est à adapter. Les noms de champs dépendent du décodeur : vérifiez-les avec `wazuh-logtest` avant d'écrire la règle, en collant un vrai événement.

Silencer, c'est une décision de sécurité. Documentez pourquoi, avec la date, dans la description de la règle.

## 2. Surveiller moins de fichiers, mais les bons

Le module d'intégrité des fichiers (`syscheck`) se configure dans `ossec.conf`. Surveiller tout le disque noie l'équipe sous les changements. Choisissez des répertoires dont la modification doit vraiment surprendre :

```xml
<syscheck>
  <directories check_all="yes" realtime="yes">/etc</directories>
  <directories check_all="yes" report_changes="yes">/root/.ssh</directories>
</syscheck>
```

`realtime="yes"` remonte le changement immédiatement, mais consomme plus de ressources : réservez-le aux dossiers sensibles et de taille raisonnable. `report_changes="yes"` conserve le contenu modifié des fichiers texte, ce qui aide à comprendre ce qui a changé.

## 3. Régler le niveau d'après l'action attendue

Une échelle simple aide à rester cohérent :

| Niveau | Ce qu'il doit déclencher |
|---|---|
| 0 | Rien : événement connu et accepté |
| 3 à 5 | Information, à consulter lors d'une recherche |
| 7 à 9 | À regarder dans la journée |
| 10 et plus | À traiter vite |

Si une règle de niveau 12 se déclenche dix fois par jour sur du trafic normal, ce n'est pas un problème de niveau 12 : c'est une règle à corriger.

## 4. Rattacher les règles à MITRE ATT&CK

Ajouter la balise `mitre` à vos règles permet de regrouper les alertes par tactique et par technique dans le tableau de bord. Cela montre aussi, sur un portfolio, que la détection est pensée par comportement et pas par signature isolée.

## 5. Tester chaque changement

Après chaque modification : `wazuh-logtest` pour voir le résultat, `wazuh-analysisd -t` pour valider la configuration, puis redémarrage. Gardez un petit fichier de lignes de journal de référence (une ligne qui doit alerter, une qui ne doit pas) et rejouez-le à chaque évolution. C'est le plus proche d'un test automatisé qu'on puisse avoir pour des règles.

## À retenir

La qualité d'une détection se mesure moins au nombre d'alertes qu'à la proportion d'alertes sur lesquelles quelqu'un agit. Chaque règle ajoutée ou silencée devrait pouvoir s'expliquer en une phrase.
