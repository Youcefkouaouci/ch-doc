---
title: Export
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [export, phpspreadsheet, xlsx, csv]
---

# Export

Architecture de la fonctionnalité d'export V3.

> Dépendance : `PHPOffice/PhpSpreadsheet` ^3.4 (licence MIT)
> Source code : `app/Jobs/Exports/GenericExport.php`, `app/Services/ExportService.php`

## Principe général

1. **Chaque modèle** implémente une méthode `headers()` retournant les en-têtes traduits liés aux clés de données :
   ```php
   return [
       __('messages.internalReference') => 'internalReference',
       __('messages.buyingDate')        => 'buyingDate',
   ];
   ```

2. **Contrôleur** : validation des champs, puis dispatch du job
3. **Job `GenericExport`** :
   - Filtrage des colonnes (depuis le stockage local côté client)
   - Génération du fichier et écriture des en-têtes
   - Appel au trait `SearchableFilterable` pour le filtrage (`searchFilterSort` / `applyFilters`)
   - Écriture des données via `ResourceCollection`
   - Envoi d'un mail avec le fichier en pièce jointe

### Colonnes exportées

Structure transmise par le client :
```json
{ "columns": ["internalReference", "buyingDate", "comment"] }
```

### Formats supportés

`.xlsx` et `.csv`

## Export des inventaires

Cas particulier :

- Structure sur du **NoSQL** (MongoDB)
- Données potentiellement très volumineuses
- Fonctionne comme un historique de détections
- Filtres personnalisés :

```json
{
  "filters": {
    "startDate": "2025-01-06 09:00:00",
    "endDate": "2025-01-06 11:00:00"
  }
}
```

> Note : `endDate` doit être postérieure à `startDate`.

Méthode dédiée : `exportInventories` dans `ExportService`.
