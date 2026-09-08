---
title: Traitement MQTT
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [iot, mqtt, tracker, capteur, payload]
---

# Traitement MQTT

Formats des messages MQTT reçus depuis les trackers Ercogener (GC81, GC06) via VerneMQ.

> Codes de statut : voir [codes-de-statut.md](codes-de-statut.md)
> Exploitation du broker : voir [Serveur MQTT](../zone-3-exploitation/runbooks/serveur-mqtt.md) (runbook)
> Source code : `app/Jobs/Treatment/MqttMessages.php`, `app/Models/Message.php`

## Trame GT (tracker)

Document MongoDB (collection `Messages`) :

```json
{
  "statusCode": 100,
  "manufacturer": "ela",
  "integrator": "charlie",
  "payload": {
    "s": "GT:2468101214161820",
    "ts": "2024-12-02T08:00:00.0Z",
    "m": "init_model",
    "loc": [23.987654, 48.987654],
    "v": {
      "ID_FRAME": { "value": "id:3" },
      "temperature": { "value": 27, "unit": "degC" },
      "battery": { "value": 80, "unit": "V", "remaining": 0 },
      "GPS": {
        "validPosition": 0,
        "speed": { "value": 0, "unit": "kmh" },
        "onMove": 16
      },
      "network": { "rsrp": -84, "rsrq": -9 }
    }
  },
  "created_at": "2024-12-04T16:09:01.369Z"
}
```

> Le numéro du tracker est extrait de `payload.s` en retirant le préfixe `GT:`.

### Champs

| Champ | Rôle |
|-------|------|
| `statusCode` | Statut de traitement de la ligne |
| `manufacturer` | Identifiant constructeur (`ela`, `teltonika`, `abeeway`) |
| `integrator` | Identifiant intégrateur (`charlie`, `charlieflespi`, `Connectic`) |
| `payload.s` | Type de trame (`GT` = tracker) + numéro |
| `payload.ts` | Date de scan du tracker |
| `payload.m` | Modèle (débogage uniquement) |
| `payload.loc` | `[latitude, longitude]` |
| `payload.v.ID_FRAME.value` | ID unique de la trame — sert à rattacher les trames capteurs (CP) |
| `payload.v.temperature` | Température (valeur + unité) |
| `payload.v.battery.remaining` | Batterie en pourcentage |
| `payload.v.GPS.onMove` | Code mouvement — voir [codes-de-statut.md](codes-de-statut.md#mouvement-tracker-onmove--mqtt) |
| `payload.v.network.rsrp / rsrq` | Qualité du signal réseau |

## Trame CP (capteur)

Document MongoDB (collection `Messages`) :

```json
{
  "statusCode": 100,
  "manufacturer": "ela",
  "integrator": "charlie",
  "payload": {
    "s": "CP:354679092915795",
    "ID_FRAME": "id:865",
    "ble_payload": [
      "P ID 01A050;0;0;-63;",
      "C ID 019D50;0;0;-84;",
      "C MOV 076541;0;0;-78;6FF5707F2500A0C095020494420303043454130"
    ]
  },
  "created_at": "2024-12-04T16:09:01.369Z"
}
```

### Champs

| Champ | Rôle |
|-------|------|
| `payload.s` | Type de trame (`CP` = capteur) + numéro du tracker |
| `payload.ID_FRAME` | Même ID que la trame GT correspondante |
| `payload.ble_payload[]` | Liste de capteurs détectés, format : `numéroCapteur;battery;battery;rssi;batteryVoltage` |
| `statusCode` | Statut de traitement |

### Format numéro capteur dans `ble_payload`

> Voir la table complète des formats dans [traitement-connect.md — Formats capteur](traitement-connect.md#formats-capteur).
