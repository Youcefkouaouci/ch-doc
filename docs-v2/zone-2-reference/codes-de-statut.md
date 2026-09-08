---
title: Codes de statut et d'état
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [référence, codes, statut, état]
---

# Codes de statut et d'état

Table de référence unique de tous les codes numériques utilisés sur la plateforme.

> **Règle** : les autres pages doivent **lier ici** au lieu de redéfinir les codes.

## Sensors / Trackers — StatusCode

<!-- BEGIN GENERATED: statuscode-sensors-trackers -->
| Code | Signification |
|------|---------------|
| 100 | Libre |
| 200 | Associé |
<!-- END GENERATED: statuscode-sensors-trackers -->

## Materials / Zones — StatusCode

<!-- BEGIN GENERATED: statuscode-materials-zones -->
| Code | Signification |
|------|---------------|
| 100 | Non connecté |
| 200 | Connecté |
| 601 | Immobile |
| 602 | Perdu |
| 603 | Retrouvé |
<!-- END GENERATED: statuscode-materials-zones -->

## Inventories — StatusCode

<!-- BEGIN GENERATED: statuscode-inventories -->
| Code | Signification |
|------|---------------|
| 100 | Non traité |
| 150 | En cours de traitement |
| 200 | Traité |
| 500 | Erreur |
<!-- END GENERATED: statuscode-inventories -->

## MqttMessage — StatusCode

<!-- BEGIN GENERATED: statuscode-mqtt -->
| Code | Signification |
|------|---------------|
| 100 | Non traité |
| 150 | En cours de traitement |
| 200 | Traité |
| 401 | Tracker inexistant |
| 402 | Trame dupliquée |
| 403 | Date invalide |
| 500 | Erreur |
<!-- END GENERATED: statuscode-mqtt -->

## Conformity — ConformityId

<!-- BEGIN GENERATED: conformity -->
| Code | Signification |
|------|---------------|
| 200 | Conforme |
| 400 | À contrôler |
| 401 | En quarantaine |
<!-- END GENERATED: conformity -->

## Alertes — Code

<!-- BEGIN GENERATED: alert-codes -->
| Code | Constante | Signification |
|------|-----------|---------------|
| 1 | `TO_CONTROL` | À contrôler |
| 2 | `QUARANTINE` | En quarantaine |
| 3 | `LOST` | Perdu |
| 4 | `MOTIONLESS` | Immobile |
| 5 | `GEOFENCING` | Geofencing |
<!-- END GENERATED: alert-codes -->

> Source code : `code-charlie/app/Models/Alert.php` (constantes `TO_CONTROL_CODE`, `QUARANTINE_CODE`, etc.)

## Types d'inventaire — TypeCode (convention 3 chiffres)

Convention : `[source][interprétation][sous-code]`

- 1er chiffre — source : `1XX` = tracker, `2XX` = téléphone, `3XX` = connect
- 2e chiffre — interprétation : `X1X` = inventaire zone, `X2X` = inventaire téléphone, `X3X` = détections
- 3e chiffre — sous-code spécifique

<!-- BEGIN GENERATED: type-codes -->
| Code | Catégorie | Signification |
|------|------------|---------------|
| 110 | Inv. zone | Inventaire tracker |
| 111 | Inv. zone | Inventaire tracker — capteur zone (un seul capteur zone pour la filiale) |
| 130 | Détection | Inventaire tracker (avec BLE/CP) sans capteur zone, ou ≥ 2 de la même filiale |
| 131 | Détection | Inventaire sans BLE/CP — tracker associé à une zone |
| 132 | Détection | Inventaire sans BLE/CP — tracker associé à un matériel |
| 210 | Inv. zone | Inventaire téléphone avec capteur zone |
| 211 | Inv. zone | Inventaire téléphone non envoyé avec capteur zone |
| 220 | Inv. téléphone | Inventaire téléphone avec capteurs ou ≥ 2 capteurs zone |
| 221 | Inv. téléphone | Inventaire QR Code |
| 230 | Détection | Inventaire téléphone non envoyé (traité comme Connect) |
| 310 | Inv. zone | Inventaire Charlie Connect avec capteur zone |
| 330 | Détection | Inventaire Charlie Connect |
| 331 | Détection | Charlie Connect communautaire |
<!-- END GENERATED: type-codes -->

## Mouvement tracker (onMove) — MQTT

<!-- BEGIN GENERATED: movement-codes -->
| Code | Label | Couleur |
|------|-------|---------|
| 16 | on_move | #2455c6 |
| 17 | static | #525252 |
| 128 | end_move | #099f1b |
| 132 | start_move | #ca6412 |
| 138 | shock | #b80f0f |
<!-- END GENERATED: movement-codes -->
