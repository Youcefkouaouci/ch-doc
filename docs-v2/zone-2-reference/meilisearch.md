---
title: Meilisearch
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [meilisearch, recherche, indexation, scout]
---

# Meilisearch

Intégration du moteur de recherche Meilisearch via Laravel Scout.

> Documentation officielle : <https://www.meilisearch.com/docs/home>
> Source code : `code-charlie/config/scout.php`, trait `SearchableFilterable`

## Setup

```bash
./dockerdo art meili:setup
```

Lance `SetupMeiliJob` (visible dans Horizon), qui exécute un `SetupMeiliModelJob` par modèle.

### En cas d'erreur

```bash
./dockerdo art db:seed --class=DeleteUnusedObjects
```

### Rafraîchir un modèle spécifique

```bash
php artisan scout:import "App\\Models\\Sensor"
```

### Réindexer sélectivement (commande dédiée)

```bash
php artisan meili:reindex Sensor Tracker Zone
```

Dispatch `SetupMeiliModelJob` pour chaque modèle spécifié.

### Accès local

`http://localhost:7700/`

### Connexion production

<À COMPLÉTER — voir LastPass pour l'URL et la clé API>

## Architecture d'implémentation

### Fichier `scout.php`

Configuration : driver `env('SCOUT_DRIVER', 'collection')` — Meilisearch en production, fallback `collection` (Eloquent) en local/test. Le trait gère ce cas via `usesEloquentFallbackDriver()`. Taille du chunk et variables d'environnement.

### Trait `SearchableFilterable`

Logique principale de recherche :

- `scopeSearchFilterSort` — scope Eloquent (s'appelle `->searchFilterSort()` sur un query builder)

### Modèles

Chaque modèle surcharge :

| Méthode | Rôle |
|---------|------|
| `toSearchableArray()` | Données recherchables, filtrables et triables — implémenté par **18 modèles** (Material, Sensor, Tracker, Zone, Site, Type, QrCode, State, File, User, Role, Subsidiary, Team, Control, ControlSheet, GpsConfig, Group, Alert) |
| `getSearchableAttributes()` | Champs pour la recherche full-text (inclut les custom fields dynamiques) |
| `getFilterableAttributes()` | Champs pour le filtrage (facettes) |
| `getSortableAttributes()` | Champs pour le tri |
| `getSearchOptions()` | Comportement spécifique (ex. : ignorer les espaces dans la recherche — Material, Sensor, Tracker, Zone) |

### Contrôleurs (View)

Les ViewControllers appellent `searchFilterSort` sur la méthode `index`.

### Observers

Mettent à jour le cache Meilisearch lors d'un changement de données.

## Problème de performance connu

L'indexation via Scout est **asynchrone** (near real-time, pas temps réel strict) :

- Les opérations d'indexation sont des tâches traitées en arrière-plan
- Les résultats de recherche peuvent ne pas être synchronisés avec la base
- Impact : incohérences visibles pour les utilisateurs après une modification
