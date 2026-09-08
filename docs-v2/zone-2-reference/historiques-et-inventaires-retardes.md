---
title: Historiques et inventaires retardés
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [historiques, inventaires, mongodb, audit]
---

# Historiques et inventaires retardés

Système d'historisation des modifications d'assets (matériel/zone/site) en MongoDB, gestion des insertions rétroactives, et cas d'inventaires retardés.

## Champs historisés

Les historiques sont générés à chaque **création, édition ou suppression** d'un élément (matériel, zone, site).

### Ce qui est affiché (onglet Activité)

- Changement sur chaque champ (réf interne, commentaire, etc.)
- Tout ce qui finit par `ID` : capteur/tracker rattaché, type, conformité, QR code, `lastMovementInventoryAssetId`
- Tout ce qui finit par `CODE` : `statusCode`
- Historiques des contrôles finalisés

### Ce qui n'est PAS affiché

- `detectionDate` des matériels et zones

## Format de stockage (MongoDB)

```json
{
  "_id": { "$oid": "67a36a641e655a5eba0ed73e" },
  "materialId": 72,
  "action": "updated",
  "source": "",
  "before": {
    "id": 72,
    "internalReference": "02BureauLille",
    "isFixed": false,
    "statusCode": "201",
    "siteId": null,
    "typeId": 1,
    "sensorId": 223,
    "trackerId": null,
    "updated_at": "2025-02-05 11:36:23"
  },
  "after": {
    "zoneId": 531,
    "updated_at": "2025-02-05 13:40:50",
    "updated_by": 212
  },
  "created_at": { "$date": "2025-02-05T13:40:52.953Z" }
}
```

- `before` : snapshot complet de l'objet avant modification
- `after` : **delta uniquement** — seuls les champs modifiés

## Insertion rétroactive d'historiques (historiques en décalé)

Quand un inventaire retardé arrive, il faut insérer un historique **entre deux existants** sans être le plus récent.

### Algorithme en 3 étapes

Soit un historique existant à 10h et un à 12h. On insère un historique à 11h :

1. L'historique à 11h **récupère le `before`** de l'historique à 12h
2. L'historique à 11h reçoit dans son `after` les modifications de 11h
3. Le `before` de l'historique à 12h est **mis à jour** avec les données de 11h

### Bug connu : `detectionSiteId`

Quand un matériel est détecté dans le même site entre deux inventaires, `after` ne contient pas `detectionSiteId` (car inchangé). Lors d'une insertion rétroactive, cela crée une **ambiguïté** : on ne sait pas dans quel site le matériel était à l'instant T.

**Proposition** : toujours inclure les données de détection (zone, site, équipe) dans les historiques à chaque inventaire, même si elles n'ont pas changé.

## Cas d'inventaires retardés

Un inventaire peut être décalé si un tracker est à court de batterie, sans réseau, ou sur un inventaire mobile sans connectivité.

> **Important** : si il n'y a pas de UPDATE, il n'y aura pas d'historique créé (les historiques sont déclenchés par les observers).

### Cas 1 : date de détection ET dernière position postérieures à la date d'inventaire

On **ne met rien à jour** — pas de geofencing ni d'alerte de mouvement.

| Entité | Champs | Action |
|--------|--------|--------|
| Current inventories | — | Pas de mise à jour |
| Sensors/Trackers | batterie | Pas de mise à jour |
| Zones | lastMovementDate, statusCode, siteId | Bloqué |
| Matériels | lastMovementDate, statusCode, siteId, teamId, zoneId | Bloqué |
| Locations | time, lat, lon, trackerId, sensorId | Bloqué |

### Cas 2 : date de détection postérieure, mais dernière position antérieure

On effectue le geofencing et l'alerte de mouvement.

| Entité | Champs | Action |
|--------|--------|--------|
| Current inventories | — | Pas de mise à jour |
| Sensors/Trackers | batterie | Pas de mise à jour |
| Zones | lastMovementDate, statusCode, siteId | Mis à jour (geofence) |
| Matériels | lastMovementDate, statusCode, siteId | Mis à jour (geofence) |
| Matériels | teamId, zoneId | Bloqué |
| Locations | time, lat, lon, trackerId, sensorId | Mis à jour (INCORRECT — devrait être bloqué) |
| Historique | time, lat, lon, trackerId, sensorId | Mis à jour |

> Pour Location, envisager d'utiliser `recordOldHistory` avec la bonne date au lieu de l'auto-observer.
