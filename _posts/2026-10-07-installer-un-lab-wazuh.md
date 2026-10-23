---
layout: post
title: "Monter un lab Wazuh en une machine virtuelle"
description: "Installer Wazuh en mode tout-en-un, récupérer les accès et enrôler un premier agent pour commencer à écrire des détections."
categories: [wazuh, lab]
---

Pour apprendre à détecter, il faut un endroit où casser des choses sans risque. Un serveur Wazuh dans une machine virtuelle suffit : il reçoit les journaux des agents, applique les règles et affiche les alertes dans un tableau de bord.

## Ce qu'il faut prévoir

- Une machine virtuelle Linux dédiée (Ubuntu Server LTS convient), avec au moins **4 vCPU et 8 Go de RAM** pour un petit lab
- Une adresse IP fixe ou une réservation DHCP, sinon les agents perdront le serveur au redémarrage
- Un accès internet pour télécharger les paquets

Le serveur, l'indexeur et le tableau de bord tournent tous sur la même machine. C'est le mode « tout-en-un », adapté à un lab et pas à une production.

## Installer le serveur

Les numéros de version changent régulièrement. Copiez toujours la commande depuis le [guide de démarrage rapide officiel](https://documentation.wazuh.com/current/quickstart.html) plutôt que depuis un article. Elle ressemble à ceci :

```bash
curl -sO https://packages.wazuh.com/<VERSION>/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

L'option `-a` demande l'installation complète sur la machine. À la fin, le script affiche l'adresse du tableau de bord, l'utilisateur `admin` et son mot de passe. Notez-le tout de suite.

Si vous l'avez manqué, il est conservé dans l'archive générée par l'installation :

```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

Ouvrez ensuite `https://<ip-du-serveur>` dans un navigateur. Le certificat est auto-signé : l'avertissement est normal dans un lab.

## Enrôler un premier agent

Dans le tableau de bord, la section des agents propose un assistant « Deploy new agent ». Choisissez le système de la machine à surveiller, saisissez l'adresse du serveur, et copiez la commande générée sur la machine cible. Puis vérifiez le service :

```bash
sudo systemctl status wazuh-agent
```

L'agent doit apparaître comme **Active** dans le tableau de bord en quelques instants.

## Où regarder quand quelque chose ne marche pas

| Fichier | Contenu |
|---|---|
| `/var/ossec/logs/ossec.log` | Journal du serveur (erreurs de configuration, connexions d'agents) |
| `/var/ossec/logs/alerts/alerts.json` | Chaque alerte générée, au format JSON |
| `/var/ossec/etc/ossec.conf` | Configuration principale du serveur |
| `/var/ossec/etc/rules/local_rules.xml` | Vos règles personnalisées |

Lire `alerts.json` directement est un bon réflexe : on y voit les champs exacts que les règles peuvent utiliser.

## Et ensuite

Une fois l'agent actif, vous pouvez générer des événements (connexions SSH échouées, modification d'un fichier sensible) et observer ce que Wazuh en fait. L'article suivant montre comment écrire une [règle de détection personnalisée]({{ '/blog/2026/10/ecrire-une-regle-de-detection-wazuh/' | relative_url }}).
