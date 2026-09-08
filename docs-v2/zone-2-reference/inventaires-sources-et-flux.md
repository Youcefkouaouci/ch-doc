---
title: "Inventaires : sources, codes et flux de données"
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [référence, inventaire, tracker, flux, mongodb]
---

# Inventaires : sources, codes et flux de données

Vue d'ensemble des sources d'inventaire, de leur pipeline et des données stockées en MongoDB.

> Codes de type détaillés : voir [codes-de-statut.md](codes-de-statut.md#types-dinventaire--typecode-convention-3-chiffres)
> Workflow visuel : voir [traitement-inventaires.md](traitement-inventaires.md)

## Identifiants manufacturer/integrator par tracker

| Tracker | manufacturer | integrator |
|---------|-------------|-----------|
| Ercogener GC81 / GC06 | `ela` | `charlie` |
| Teltonika GC234 | `teltonika` | `charlieflespi` ou `Connectic` |
| Digital Matter TCOY3G | `teltonika` | `Connectic` |
| Abeeway Compact | `abeeway` | `charlie` |

## Matrice de synthèse par source

| Source | Entrée | Mongo d'abord | SQL direct | Types | Collections/tables impactées |
|--------|--------|:---:|:---:|-------|-----|
| Charlie Connect | `/api/monitoring` ou `/api/v2/monitoringMessages` | Oui | Non | 310, 330, 331 | Mongo `monitoring`, SQL `inventories`, `inventories_assets`, `materials`, `zones`, `locations`, histories |
| Charlie Gestion Mobile | API REST mobile | Non | Oui | 210, 211, 220, 221, 230 | SQL `inventories`, `inventories_assets`, `materials`, `zones`, `locations`, histories |
| Ercogener (MQTT) | VerneMQ | Oui | Non | 110, 111, 130, 131, 132 | Mongo `mqttMessages`, SQL inventaires/assets, histories |
| Teltonika | Flespi → endpoint | Oui | Non | 110, 130, 131, 132 | Mongo message brut, SQL inventaires/assets, histories |
| Digital Matter | Flespi → endpoint | Oui | Non | 110, 130, 131, 132 | Mongo message brut, SQL inventaires/assets, histories |
| Abeeway | Endpoint Charlie | Oui | Non | 110, 130, 131, 132 | Mongo message brut, SQL inventaires/assets, histories |

## Tableau des cas d'inventaire

| N° | Cas | Type | Code |
|----|-----|------|------|
| 1.1 | Tracker (Teltonika/Abeeway/GC81/GC06) associé à 1 zone, pas de BLE | Détection | 131 |
| 1.2 | Tracker GPS associé à un matériel, trame GT sans CP | Détection | 132 |
| 2 | Matériel avec tracker GC234/GC81/GC06, trame BLE | Détection | 130 |
| 3 | Zone avec tracker GC234/GC81/GC06/Abeeway, trame BLE | Inventaire | 110 |
| 4 | Charlie Connect classique | Détection | 330 |
| 5 | Zone avec un seul capteur zone, détecté par Connect | Inventaire | 310 |
| 6 | Charlie Connect communautaire | Détection | 331 |
| 7 | Scan QR Code | Détection | 221 |
| 8 | Scan téléphone manuel (sans capteur zone) | Inventaire | 220 |
| 9 | Scan téléphone manuel (avec capteur zone) | Inventaire | 210 |
| 10 | Scan téléphone non envoyé (sans capteur zone) | Détection | 230 |
| 11 | Scan téléphone non envoyé (avec capteur zone) | Inventaire | 211 |
| 12 | Zone avec capteur zone, détecté par tracker d'une filiale | Inventaire | 111 |

## Swimlanes par source

### Charlie Connect

Application de monitoring BLE/beacons, fonctionne en continu, détecte les entrées/sorties de régions BLE. Endpoints : `/api/v2/monitoringMessages` (standard) et `/api/monitoring` (signature RSA SHA-256).

> Payload détaillé : voir [traitement-connect.md](traitement-connect.md)

### Charlie Gestion Mobile

Exception structurante : le traitement **ne passe pas par Mongo**. Application Flutter, API REST, offline-first. Les événements terrain sont stockés localement et rejoués en cas d'échec réseau.

### Ercogener (MQTT via VerneMQ)

Flux IoT via VerneMQ → routage vers les workloads. `manufacturer = ela`, `integrator = charlie`.

> Payload détaillé : voir [traitement-mqtt.md](traitement-mqtt.md)

### Teltonika / Digital Matter (via Flespi)

Arrivée via Flespi Panel. Teltonika GC234 : `manufacturer = teltonika`, `integrator = charlieflespi` ou `Connectic`. Digital Matter TCOY3G : `manufacturer = teltonika`, `integrator = Connectic`.

### Abeeway (via endpoint)

Arrivée via endpoint. `manufacturer = abeeway`, `integrator = charlie`. Traitement identique aux autres trackers (Mongo → normalisation → inventaire SQL).

## Exemple payload Abeeway (MongoDB)

> Payload complet d'une trame Abeeway Compact Tracker avec BLE beacons — stockage MongoDB.

Champs clés du document :

| Champ | Rôle |
|-------|------|
| `payload.device_e_u_i` | Device EUI du tracker |
| `payload.coordinates` | `[longitude, latitude, altitude]` |
| `payload.resolved_tracker.dynamicMotionState` | `MOVING` / `STATIC` |
| `payload.resolved_tracker.batteryLevel` | Niveau de batterie (%) |
| `payload.resolved_tracker.temperatureMeasure` | Température |
| `payload.resolved_tracker.scanCollection.macAddressData[]` | Liste BLE beacons détectés (`mac` + `rssi`) |
| `payload.downlink_url` | URL Actility pour le downlink |
| `statusCode` | Statut de traitement |

> Le payload complet est volumineux (~700 lignes). Consulter un document MongoDB réel pour le détail. Source originale : `notion-doc/Sans titre/Général`.
