---
title: Traitement Charlie Connect
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [iot, connect, payload, capteur]
---

# Traitement Charlie Connect

Format des données remontées par Charlie Connect (et communautaire) via les endpoints API.

> Codes de statut : voir [codes-de-statut.md](codes-de-statut.md)
> Endpoint : `POST /api/monitoring` (signature RSA SHA-256 via header `X-Signature`, middleware `VerifyRSASignature`)

## Trame brute (avec GPS)

```json
{
  "manufacturer": "charlieConnectAndroid",
  "integrator": "Charlie",
  "payload": {
    "jsonVersion": 1,
    "appVersion": "2.4.0",
    "timestampRanging": "2025-11-19T15:18:41.137Z",
    "timestampInventory": "2025-11-19T15:20:57.587997Z",
    "isCommunautary": false,
    "informations": [
      {
        "number": "C ID 00BDB7",
        "latitude": 49.23034627921879,
        "longitude": 2.8916223999112844,
        "detectionDate": "2025-11-19T15:19:12.379Z",
        "timestampGps": "2025-11-19T15:19:05.607Z"
      }
    ]
  }
}
```

## Champs de la trame

| Champ | Rôle |
|-------|------|
| `manufacturer` | Identifiant constructeur (`charlieConnectAndroid`) |
| `integrator` | Identifiant intégrateur (`Charlie`) |
| `payload.jsonVersion` | Version du format JSON |
| `payload.appVersion` | Version de l'application (débogage) |
| `payload.timestampRanging` | Timestamp de début du ranging (débogage) |
| `payload.timestampInventory` | Timestamp de l'appel API (débogage) |
| `payload.isCommunautary` | `true` si trame du réseau communautaire |
| `statusCode` | Statut de traitement de la ligne |

## Champs par capteur (`informations[]`)

| Champ | Rôle |
|-------|------|
| `number` | Numéro du capteur — voir [Formats capteur](#formats-capteur) ci-dessous |
| `detectionDate` | Dernière date de détection sur l'antenne |
| `timestampGps` | Timestamp de la dernière localisation acquise |
| `latitude` / `longitude` | Position rattachée au capteur |
| `batteryVoltage` | Voltage en millivolts (mV) |
| `battery[Value]` | Batterie en pourcentage |
| `nbFrames` | Nombre de frames (capteurs MOV uniquement) |
| `nbMoves` | Nombre de fronts montants au-dessus du seuil d'accélération (capteurs MOV uniquement) |

## Variantes du payload `informations`

**Avec GPS** — contient `latitude`, `longitude`, `timestampGps` par capteur.

**Sans GPS** — seuls `detectionDate` et `batteryVoltage` sont présents.

## Formats capteur

Table de référence des préfixes de numéro capteur, définis dans `app/Models/Format.php`.

<!-- BEGIN GENERATED: sensor-formats -->
| Préfixe | Type |
|----------|------|
| `C ID` | COMPACT LED |
| `L ID` | LITE LED |
| `P ID` | MAX LED |
| `R ID` | LITE BASIC |
| `T ID` | MAX BASIC |
| `A ID` | EYE BEACON |
<!-- END GENERATED: sensor-formats -->

> `C MOV` peut encore apparaître dans des payloads legacy, mais a été **supprimé** des formats enregistrés (`CreateFormatsJob` force-delete).

## Clés RSA

Les clés RSA (production et test) sont stockées dans **LastPass**. Ne pas les inclure dans le code ou la documentation.
