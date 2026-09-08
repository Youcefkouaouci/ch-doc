---
title: Configuration GPS et anonymisation RGPD
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [gps, rgpd, anonymisation, mongodb]
---

# Configuration GPS et anonymisation RGPD

Configuration de la désactivation GPS (périodes sans géolocalisation) et règles d'anonymisation RGPD appliquées lors du traitement des inventaires.

> Maquettes : [Figma — Config GPS](https://www.figma.com/design/LLLbUGnbqSBuXPDJLr4rMT/Charlie-Solution?node-id=373-6319)

## Types de configuration

Trois types de récurrence, stockés en **MongoDB**.

### Ponctuel — Code 1

Désactivation pour une période spécifique (ex. : vacances).

```json
{
  "id": 1,
  "name": "ponctuel",
  "code": 1,
  "subsidiaryId": 1,
  "isActive": true,
  "zoneIds": [1, 2],
  "startDate": "2025-03-03 08:00:00",
  "endDate": "2025-03-07 20:00:00"
}
```

### Intervalle — Code 2

Désactivation avec heures fixes sur plusieurs jours (ex. : vendredi 18h → lundi 8h, toutes les semaines).

```json
{
  "id": 1,
  "name": "intervalle",
  "code": 2,
  "subsidiaryId": 1,
  "isActive": true,
  "zoneIds": [1, 2],
  "period": {
    "start": { "day": 4, "hour": "18:00" },
    "end":   { "day": 0, "hour": "08:00" }
  },
  "frequency": 168,
  "startDate": "2025-03-07 18:00:00",
  "endDate": null
}
```

| Champ | Rôle |
|-------|------|
| `period.start.day` / `period.end.day` | Jour de la semaine (0 = Lundi, 6 = Dimanche) — **convention non-standard** (ISO : Lundi = 1) |
| `frequency` | Nombre d'heures entre chaque récurrence (168 = une semaine) |
| `startDate` | Doit correspondre au même jour/heure que `period.start` |
| `endDate` | `null` = pas de fin |

### Jours fixes — Code 3

Désactivation avec heures fixes sur des journées précises (ex. : lundis, samedis, dimanches de 8h à 18h).

```json
{
  "id": 1,
  "name": "jours fixes",
  "code": 3,
  "subsidiaryId": 1,
  "isActive": true,
  "zoneIds": [1, 2],
  "days": [0, 5, 6],
  "startHour": "08:00",
  "endHour": "18:00",
  "startDate": "2025-03-05 08:00:00",
  "endDate": null
}
```

| Champ | Rôle |
|-------|------|
| `days` | Jours de la semaine (0 = Lundi, 6 = Dimanche) |

## Règles d'anonymisation RGPD

### Règle 1 — Toutes les sources d'inventaire

Pour chaque zone, si l'heure de l'inventaire se trouve dans une configuration GPS **active** de la zone → `latitude` et `longitude` de la zone sont mises à **null**.

### Règle 2 — Inventaires GT uniquement

Concerne : MQTT GT Tracker, MQTT GT Sensor, Connect GT, Phone GT, Phone Connect GT.

Si la zone devant être anonymisée est liée à un **tracker**, OU si c'est la **seule zone** de l'inventaire → `latitude` et `longitude` de **tous les capteurs** sont mises à **null**.
