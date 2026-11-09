---
layout: post
title: "Wazuh : réduire le bruit pour garder les alertes utiles"
description: "Cinq habitudes pour qu'un tableau de bord Wazuh reste lisible : silencer proprement, surveiller peu mais bien, tester chaque changement."
categories: [wazuh]
---

Un Wazuh fraîchement installé produit beaucoup d'alertes. Si tout est important,l'analyste s'épuise à classifier des faux positifs Voici cinq étapes pour garder un signal propre et exploitable.

## 1. Réduire au silence une règle sans la supprimer

Il ne faut pas modifier les fichiers de règles officiels. Ils sont écrasés à chaque mise à jour de Wazuh. Il faut Écrire les règles au niveau 0 dans `local_rules.xml`. Car un niveau 0 ne génère aucune alerte.
Voici un exemple sur le dossier C:\Program Files:
```xml
<group name="sysmon">
<rule id="100012" level="3">
<if_sid>92213</if_sid>
<field name="win.eventdata.image" type="pcre2">(?i)^C:\\Program Files( \(x86\))?\\</field>
<description>False Positive DLL in Temp create by a binarie install on (Program Files)</description>
</rule>
```

Ce modèle est à adapter. Les noms de champs dépendent du décodeur. Il faut vérifier avec l'outil `wazuh-logtest` avant d'écrire la règle, en collant un vrai événement.

Réduire au silence est une décision de sécurité, il est important de Documenter pourquoi, avec la date, et mettre dans la documentation le bout de la règle utiliser.

## 2. Surveiller moins de fichiers, mais les bons

Le module d'intégrité des fichiers (`syscheck`) se configure dans `ossec.conf`. Surveiller tout le disque noie l'équipe sous les changements. Il est important de choisir des répertoires dont la modification doit vraiment surprendre :

Voici un exemple de réglage pour des postes Windows:
```xml
<syscheck>
  <directories check_all="yes" realtime="yes">C:\Windows\System32</directories>
  <directories check_all="yes" report_changes="yes">C:\Windows\SysWOW64</directories>
</syscheck>
```

`realtime="yes"` remonte le changement immédiatement, mais consomme plus de ressources. Il faut le réserver aux dossiers sensibles et de taille raisonnable. `report_changes="yes"` conserve le contenu modifié des fichiers texte, ce qui aide à comprendre ce qui a changé.

Bien sur cela est un extrait d'exemple il y a plusieurs autres sources à surveiller sur un poste Windows qui pourrais être susceptible d'intéresser des attaquants (nous pourrons voir cela dans un autre billet de blog)
## 3. Régler le niveau d'après l'action attendue

Voici mon interprétation des différents niveau d'alerte présent dans Wazuh :

| Niveau | Ce qu'il doit déclencher |
|---|---|
| 0 | Rien : événement connu et accepté |
| 3 à 5 | Information, à consulter lors d'une recherche |
| 7 à 9 | À regarder dans la journée |
| 10 et plus | À traiter vite |

Si une règle de niveau 12 se déclenche dix fois par jour sur du trafic normal, ce n'est pas un problème de niveau 12 : c'est une règle à corriger et à voir si c'est un événement connu et accepté au sein du SI

## 4. Rattacher les règles à MITRE ATT&CK

Une bonne pratique à mettre en place est d'ajouter la balise `mitre` à vos règles, ce qui permet de regrouper les alertes par tactique et par technique dans le tableau de bord. Afin que la détection soit  pensée par comportement et pas par signature isolée.

## 5. Tester chaque changement

Après chaque modification -> `wazuh-logtest` pour voir le résultat, `wazuh-analysisd -t` -> pour valider la configuration, puis redémarrage. Il est important de garder un petit fichier de lignes de journal de référence (une ligne qui doit alerter, une qui ne doit pas) et rejouez-le à chaque évolution. C'est le plus proche d'un test automatisé qu'on puisse avoir pour des règles.

## À retenir

La qualité d'une détection se mesure moins au nombre d'alertes qu'à la proportion d'alertes sur lesquelles quelqu'un agit. Chaque règle ajoutée ou réduit au silence doit pouvoir être expliqué en une phrase.

Merci d'avoir lu se premier billet de blog la tâche n'est pas aisé c'est une grande première pour moi de partager ainsi ce que je fait de mon côté. 
Le log s'éttofera au fur et à mesure du temps que je peu allouer à la rédaction. Bien sur il n y auras pas que du Wazuh mais tout l'aspect blue team que je vous partegerais ainsi au gré de mon évolution dans le sujet.
