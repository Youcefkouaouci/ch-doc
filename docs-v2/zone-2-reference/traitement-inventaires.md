---
title: Traitement des inventaires
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [iot, inventaire, traitement, workflow]
---

# Traitement des inventaires

Workflow de traitement des inventaires, convention des codes et format de stockage.

> Workflow visuel : [Figma — TreatmentInventories](https://www.figma.com/board/iVEnZGbDftWBGu1vewxLli/TreatmentInventories)
> Codes de type d'inventaire : voir [codes-de-statut.md](codes-de-statut.md#types-dinventaire--typecode-convention-3-chiffres)

## Convention de nommage des TypeCodes

Les codes sont basés sur 3 chiffres : `[source][interprétation][sous-code]`

- **1er chiffre** — source : `1XX` = tracker, `2XX` = téléphone, `3XX` = connect
- **2e chiffre** — interprétation : `X1X` = inventaire zone, `X2X` = inventaire téléphone, `X3X` = détections
- **3e chiffre** — sous-code spécifique

### Inventaire traité sur une zone ciblée

| Code | Description |
|------|------------|
| 110 | Inventaire tracker |
| 111 | Inventaire tracker — capteur zone (un seul capteur zone pour la filiale) |
| 310 | Inventaire Charlie Connect avec capteur zone |
| 210 | Inventaire téléphone avec capteur zone |
| 211 | Inventaire téléphone non envoyé avec capteur zone |

### Inventaire traité comme un Charlie Connect

| Code | Description |
|------|------------|
| 130 | Inventaire tracker (BLE/CP) sans capteur zone, ou ≥ 2 même filiale |
| 330 | Inventaire Charlie Connect |
| 331 | Charlie Connect communautaire |
| 230 | Inventaire téléphone non envoyé |

### Inventaire téléphone

| Code | Description |
|------|------------|
| 220 | Inventaire téléphone avec capteurs ou ≥ 2 capteurs zone |
| 221 | Inventaire QR Code |

### Détections (sans BLE / sans trames capteurs)

| Code | Description |
|------|------------|
| 131 | Tracker associé à une zone |
| 132 | Tracker associé à un matériel |

## Format de stockage

### Table `inventories`

Paramètres communs à un inventaire :

| Champ | Rôle |
|-------|------|
| zoneId / userId | Zone de détection ou utilisateur (inventaire téléphone) |
| time | Date de réalisation |
| statusCode | Traité ou non (voir [codes-de-statut.md](codes-de-statut.md#inventories--messages--statuscode)) |
| typeOfInventory | Type d'inventaire (voir ci-dessus) |
| movementStatus | État de mouvement (optionnel) : démarrage, arrêt, choc |
| detectionTeamId | Équipe de détection |
| subsidiaryId | Filiale liée à la zone du tracker/capteur |
| messageId | Référence vers le message source (ajouté en 2026-06) |
| created_at | Date d'insertion physique (peut différer de `time`) |

### Table `inventories_assets`

Liste des assets détectés dans un inventaire :

| Champ | Rôle |
|-------|------|
| inventoryId | Référence vers `inventories` |
| typeId | Type de l'asset |
| materialId / zoneId | L'un des deux est rempli (matériel ou zone) |
| detectionSiteId | Site de détection |
| detectionDate | Date de détection (peut varier par capteur sur Connect) |
| latitude / longitude | Position de l'asset |
| battery | Batterie éventuelle |
| rssi | Valeur RSSI (dBm) |
| nbMoves | Nombre de fronts montants (capteurs MOV) |
| nbFrames | Nombre de frames (capteurs MOV) |
| created_at | Date d'insertion physique |
