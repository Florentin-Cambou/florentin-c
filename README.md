# Portfolio et blog cybersécurité

Site statique Jekyll, hébergé gratuitement par GitHub Pages. Vous écrivez en Markdown, GitHub construit et publie le site à chaque modification.

## Mise en ligne

1. Sur GitHub, créez un dépôt **public** nommé `VOTRE-PSEUDO.github.io` (c'est ce nom exact qui donne l'adresse `https://VOTRE-PSEUDO.github.io`).
2. Envoyez-y le contenu de ce dossier :
   ```bash
   git init
   git add .
   git commit -m "Premier commit du portfolio"
   git branch -M main
   git remote add origin https://github.com/VOTRE-PSEUDO/VOTRE-PSEUDO.github.io.git
   git push -u origin main
   ```
3. Dans le dépôt : **Settings → Pages**. Sous « Build and deployment », choisissez **Deploy from a branch**, branche `main`, dossier `/ (root)`, puis enregistrez.
4. Patientez une ou deux minutes. Le site apparaît à l'adresse annoncée en haut de la page Pages.

## À modifier en premier

Ouvrez `_config.yml` et remplacez :

- `title`, `author`, `tagline` : votre nom et votre accroche
- `url` : `https://VOTRE-PSEUDO.github.io`
- `github`, `linkedin`, `htb`, `email` : vos identifiants (un champ vide masque le lien)

Puis relisez `index.html`, `projets.md` et `a-propos.md` pour y mettre vos informations réelles.

## Publier un article

Créez un fichier dans `_posts/` nommé `AAAA-MM-JJ-titre-de-l-article.md`, avec ce début :

```yaml
---
layout: post
title: "Mon titre"
description: "Une phrase de résumé."
categories: [wazuh]
---
```

Le modèle `_drafts/modele-article.md` donne une structure à suivre. Une date dans le futur masque l'article jusqu'à ce jour-là.

Pour ajouter une image : placez-la dans `assets/img/` et écrivez `![Description]({{ '/assets/img/capture.png' | relative_url }})`. Floutez les adresses IP, noms de machines et mots de passe réels avant de publier une capture.

## Tester en local (facultatif)

```bash
bundle install
bundle exec jekyll serve
```

Le site est alors visible sur `http://localhost:4000`.

## Bonnes pratiques de publication

- Ne publiez pas de solutions pour les machines **actives** de Hack The Box : leurs règles l'interdisent. Les machines retirées et la méthodologie générale sont possibles.
- Ne copiez pas le contenu des modules payants de l'Academy.
- Ne publiez jamais de secrets : clés, mots de passe, jetons, adresses internes.
- Les trois articles fournis sont des exemples techniques. Remplacez-les ou enrichissez-les avec vos propres captures et résultats.
