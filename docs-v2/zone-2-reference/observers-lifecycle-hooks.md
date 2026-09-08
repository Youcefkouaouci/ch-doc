---
title: Observers et hooks de cycle de vie
status: needs-update
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [observers, lifecycle, geocoding, geofencing, sync]
---

{% hint style="warning" %}
**Page en cours de vérification** — l'intégration SimplicitiService n'est pas documentée (voir `[CODE?]` §Matériels).
{% endhint %}

# Observers et hooks de cycle de vie

Architecture des side-effects lors de la création, modification et suppression des entités principales.

> Source code : `app/Observers/`, `app/Jobs/`, `app/Managers/LocationManager.php`

## Architecture : qui fait quoi

> **Correction Phase 3** — le geocoding et le geofencing ne sont **pas** dans les observers. Ils sont dans les **contrôleurs** (via `LocationManager::updateOrCreateFixedLocation()`). Les observers gèrent les side-effects post-persist : statuts capteurs/trackers, historiques MongoDB, sync Meilisearch, et dispatch de jobs.

| Action | Où ça se passe | Fichiers clés |
|--------|---------------|---------------|
| Geocoding (adresse → lat/lon ou inverse) | **Contrôleur** | `LocationManager::updateOrCreateFixedLocation()` |
| Geofencing (site → assignation auto) | **Contrôleur** | `LocationManager`, appelé depuis `ZonesController`, `MaterialsController` |
| Création/update localisation | **Contrôleur** | `LocationManager` |
| Mise à jour statut capteur/tracker | **Observer → Job** | `AfterCreate`, `AfterUpdate`, `AfterDelete` |
| Historisation (MongoDB) | **Observer** | `MaterialsHistories::record()`, `ZonesHistories::record()`, etc. |
| Sync Meilisearch | **Observer** | Appel `->searchable()` sur les relations liées |
| Sync plateforme interne | **Observer → Job** | Jobs `Sensors/`, `Trackers/`, `Materials/` |

## Zones

### Observer (`ZonesObservers`)

**created** : dispatch `Jobs\Zones\AfterCreate` (met le statut capteur/tracker à ASSOCIATED) + enregistre l'historique.

**updated** : dispatch `Jobs\Zones\AfterUpdate` (swap statut si changement capteur/tracker) + sync Meilisearch des matériels/trackers/capteurs/QR codes liés + historique.

**deleted** : dispatch `Jobs\Zones\AfterDelete`.

### Contrôleur (`ZonesController`)

Avant le save : appelle `LocationManager` pour le geocoding et le geofencing. Le site géofencé **prime** sur le site renseigné manuellement.

## Matériels

### Observer (`MaterialsObservers`)

**created** : dispatch `Jobs\Materials\AfterCreate` (statut capteur/tracker + **SimplicitiService::create()** pour sync plateforme interne containers) + historique.

**updated** : dispatch `Jobs\Materials\AfterUpdate` (swap statut + **SimplicitiService::update()**) + sync Meilisearch + historique.

**deleted** : dispatch `Jobs\Materials\AfterDelete`.

> **`[CODE?]` SimplicitiService** — intégration avec la plateforme interne pour les containers. Non documentée auparavant. Nécessite un owner et probablement un ADR.

### Contrôleur (`MaterialsController`)

Même pattern que Zones : `LocationManager` pour geocoding/geofencing avant le save.

## Capteurs

### Observer (`SensorsObserver`)

**created** : dispatch job → `SensorManager::linkSensorOnInternalPlatform` (association au client sur la plateforme interne).

**updated** : dispatch job → envoie **tous les champs modifiés** (battery, statusCode, number, macId, major, minor, uuid) via `updateInternalSensor`. Si le numéro change, `oldNumber` est transmis.

**deleted** : dispatch job → désassociation sur la plateforme interne + **clear `sensorId` sur l'asset lié** + passage à `NOT_EQUIPPED`.

## Trackers

### Observer (`TrackersObserver`)

**created** : dispatch job → `TrackerManager::linkTrackerOnInternalPlatform`.

**updated** : dispatch job → envoie tous les champs modifiés via `updateInternalTracker`.

**deleted** : dispatch job → désassociation + **clear `trackerId` sur l'asset lié** + passage à `NOT_EQUIPPED`.

## Autres observers (20 au total)

Le dossier `app/Observers/` contient **20 fichiers**. Les 16 non détaillés ci-dessus :

| Observer | Entité |
|----------|--------|
| `BrandsObservers` | Brand |
| `ClientsObservers` | Client |
| `ControlsObservers` | Control |
| `CustomFieldsObservers` | CustomField |
| `EavsObservers` | EAV |
| `FilesObservers` | File |
| `GroupsObservers` | Group |
| `LocationsObservers` | Location |
| `ModelsObservers` | Model (device model) |
| `PeriodicitiesObservers` | Periodicity |
| `SitesObservers` | Site |
| `StatesObservers` | State |
| `SubsidiariesObservers` | Subsidiary |
| `TeamsObservers` | Team |
| `TypesObservers` | Type |
| `UsersObserver` | User |

> Ces observers suivent le même pattern (historisation + sync Meilisearch). Le code est la doc pour le détail de chacun.

## Points d'attention

- **Changement capteur/tracker** sur un matériel ou une zone : 3 cas à gérer (ajout, retrait, remplacement)
- Toutes les synchronisations vers la plateforme interne se font dans des **jobs** (queues)
- Le geofencing s'applique dès qu'une adresse/géolocalisation est ajoutée (dans le contrôleur, pas dans l'observer)
