---
title: Commandes utiles
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [dev, commandes, sql, mongodb, meilisearch]
---

# Commandes utiles

## Métier

### Regénérer la documentation API (Swagger)

```bash
./dockerdo art l5-swagger:generate
```

URL locale : `localhost/api/documentation`

### Fiche de contrôle non générée

Si un contrôle a un `fileId` nul mais un `controlFields` rempli (et n'est pas en brouillon) :

1. Se connecter sur le pod worker du namespace
2. Lancer les commandes suivantes (remplacer `<ID>` par l'ID du contrôle) :

```php
php artisan tinker

use App\Jobs\Controls;
AfterCreate(<ID>)::dispatch();
```

Exécuter la dernière commande **2 fois**. Vérifier via Horizon que le job est passé et que `fileId` est renseigné dans la table `controls`.

### Réindexation d'un modèle sur Meilisearch

```php
Model::where(...)->searchable()
```

> Voir aussi : [zone-2/meilisearch.md](../zone-2-reference/meilisearch.md)

## Bases de données

### Requête SQL : taille des DB MySQL

```sql
SELECT
    table_schema AS database_name,
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS size_mb,
    ROUND(SUM(data_length + index_length) / 1024 / 1024 / 1024, 2) AS size_gb
FROM information_schema.tables
GROUP BY table_schema
ORDER BY size_mb DESC;
```

### Requête SQL : pourcentage de connexions utilisées

```sql
SELECT
  @@max_connections AS max_connections,
  VARIABLE_VALUE AS threads_connected,
  ROUND(VARIABLE_VALUE / @@max_connections * 100, 2) AS usage_percent
FROM performance_schema.global_status
WHERE VARIABLE_NAME = 'Threads_connected';
```

### Requête SQL : détail des connexions actuelles

```sql
SELECT
  USER,
  HOST,
  DB,
  COMMAND,
  COUNT(*) AS connections
FROM information_schema.PROCESSLIST
GROUP BY USER, HOST, DB, COMMAND
ORDER BY connections DESC;
```

### Requête SQL : nombre d'inventaires par type de matériel

```sql
SELECT COUNT(DISTINCT ia.inventoryId) AS inventories_count
FROM inventories_assets ia
LEFT JOIN materials m ON m.id = ia.materialId
WHERE m.typeId = 134;
```

### Requête Compass shell : taille des DB MongoDB

```javascript
db.adminCommand({ listDatabases: 1 }).databases.map(d => ({
  db: d.name,
  poids_Mo: Math.round(d.sizeOnDisk / 1024 / 1024 * 100) / 100
}))
```

### Requête MongoDB : récupérer des données sur une plage horaire

```json
{
  "created_at": {
    "$gte": "ISODate('2026-04-09T16:30:00Z')",
    "$lte": "ISODate('2026-04-09T18:00:00Z')"
  },
  "payload.deviceId": "feb230fcf3cc57fe"
}
```

### Filtres MongoDB Compass

#### Opérateurs de base

| Opérateur | Usage |
|-----------|-------|
| `$lt` / `$lte` | Inférieur (strict / ou égal) |
| `$exists` | Vérifier l'existence d'une clé |
| `$ne: []` | Vérifier qu'un tableau n'est pas vide |

Exemple — vérifier la présence de BLE beacons :

```json
{
  "payload.ble_beacons": {
    "$exists": true,
    "$ne": []
  }
}
```

#### Pipeline d'agrégation : détecter les doublons BLE beacons

> Sélectionner l'option **TEXT** (pas STAGES) dans l'onglet Aggregations de Compass.

```json
[
  {
    "$match": {
      "payload.timestamp": { "$exists": true },
      "payload.ble_beacons": { "$exists": true, "$ne": [] }
    }
  },
  { "$unwind": "$payload.ble_beacons" },
  {
    "$group": {
      "_id": {
        "ident": "$payload.ident",
        "timestamp": "$payload.timestamp",
        "beacon_id": "$payload.ble_beacons.id"
      },
      "occurrences": { "$sum": 1 },
      "records": {
        "$push": {
          "document_id": "$_id",
          "record_seqnum": "$payload.record_seqnum",
          "created_at": "$created_at",
          "server_timestamp": "$payload.server_timestamp",
          "rssi": "$payload.ble_beacons.rssi",
          "state": "$payload.ble_beacons.state"
        }
      }
    }
  },
  { "$match": { "occurrences": { "$gt": 1 } } },
  {
    "$group": {
      "_id": {
        "ident": "$_id.ident",
        "timestamp": "$_id.timestamp"
      },
      "duplicate_beacons": {
        "$push": {
          "beacon_id": "$_id.beacon_id",
          "occurrences": "$occurrences",
          "records": "$records"
        }
      },
      "duplicate_count": { "$sum": 1 }
    }
  },
  { "$sort": { "_id.timestamp": -1 } }
]
```

Le résultat se trouve à droite, sous **Pipeline Output**.

### Erreur création de BDD MySQL

En cas de problème de droits lors de la création d'une DB MySQL :

```bash
# Forcer la connexion avec root
./vendor/bin/sail exec mysql mysql -uroot -ppassword
```

```sql
-- Afficher les utilisateurs
SELECT USER(), CURRENT_USER();
-- Créer la DB
CREATE DATABASE IF NOT EXISTS `dev`;
-- Donner les droits
GRANT ALL PRIVILEGES ON `dev`.* TO 'sail'@'%';
-- Appliquer
FLUSH PRIVILEGES;
```

## Linux / Docker

### Conflit port 1883 (MQTT)

Au lancement du Docker, si le port 1883 est déjà utilisé :

```bash
sudo service mosquitto stop
```

### Extraction de logs par date

```bash
grep '^\[2026-03-19' storage/logs/laravel.log
```

## NPM / Vite

### Erreur fichier non existant dans `.vite/deps`

1. Stopper `npm run` si en cours
2. `rm -rf node_modules/.vite`
3. Relancer `npm run dev`

## Commandes infra

> <À COMPLÉTER> — cette section est un placeholder pour les commandes infra (ancienne page « Commandes utiles infra » — vide à ce jour).
