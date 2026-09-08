---
title: Contrôles — Fiches et templates
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [contrôles, templates, json, fiches]
---

# Contrôles — Fiches et templates

Processus de création et d'intégration de nouvelles fiches de contrôle (templates JSON).

> Source code : `code-charlie/app/Jobs/Migrations/CreateControlTemplatesJob.php`

## Processus d'ajout d'une nouvelle fiche

1. **Créer le JSON** de la fiche en suivant la structure V3
2. **Ajouter les traductions** dans `lang/controlTemplates/translations.json`
3. **Référencer le JSON** dans `CreateControlTemplatesJob.php` :

```php
'general' => [
    ControlTemplate::DEFAULT_CONTROL_TEMPLATE_ID => [
        'name'         => 'FICHE DE CONTRÔLE GÉNÉRALE',
        'jsonFilePath' => '/templates/general_control_template.json',
    ],
],
// Key = partie de l'APP_URL du client
'etf' => [
    2 => [
        'name'         => 'FICHE DE CONTRÔLE TIREFONNEUSE',
        'jsonFilePath' => '/templates/etf/tirefonneuses.json',
    ],
],
```

4. **Ajouter dans le seeder** : `database/seeders/FixControlTemplateIds.php`
5. **Ajouter la traduction du `name`** dans `lang/*/messages.php`
6. **Exécuter** :
   ```bash
   ./dockerdo art db:seed --class=ControlTemplatesSeeder
   ```
7. **Lier le template** à un type de matériel, tester la création/édition (images, champs)

## Ciblage par client

- Clé `general` → tous les clients
- Clé spécifique (ex. `etf`) → partie de l'`APP_URL` du client (doit être unique)
- Option `subsidiaryIds` → cibler des filiales spécifiques (par défaut : toutes)

## Sous-pages de référence

Les pages suivantes étaient liées dans Notion mais non exportées avec contenu :

- Structure JSON V3
- Control Fields
- État des lieux
- Fonctionnement
- Tables BDD

> <À COMPLÉTER> — ces sous-pages n'étaient pas dans l'export Notion. Si elles existent, les migrer ici comme sections.
