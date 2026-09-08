---
title: Documentation technique — Charlie Platform V3
status: current
owner: "<à assigner>"
last_reviewed: 2026-09-08
review_every: 3m
tags: [accueil, navigation]
---

# Documentation technique — Charlie Platform V3

Documentation de référence de la plateforme Charlie (V3). Organisée en 4 zones selon l'usage.

## Zones

### [Zone 1 — Coder au quotidien](zone-1-coder-au-quotidien/acces-externes.md)

Démarrer sur le projet : accès, base de données locale, commandes, seeders.

- [Accès externes](zone-1-coder-au-quotidien/acces-externes.md)
- [Commandes utiles](zone-1-coder-au-quotidien/commandes-utiles.md)
- [Import BDD client en local](zone-1-coder-au-quotidien/import-bdd-local.md)
- [Seeders et rôles](zone-1-coder-au-quotidien/seeders-et-roles.md)

### [Zone 2 — Référence plateforme](zone-2-reference/codes-de-statut.md)

Comment marche la plateforme : traitements IoT, alertes, EAV, recherche, API, observers, exports, contrôles. Suit le découpage du code source.

15 pages couvrant : [alertes](zone-2-reference/alertes-et-notifications.md), [API validation](zone-2-reference/api-validation-metier.md), [codes de statut](zone-2-reference/codes-de-statut.md), [GPS/RGPD](zone-2-reference/configuration-gps-anonymisation.md), [contrôles](zone-2-reference/controles-fiches-templates.md), [EAV](zone-2-reference/eav-champs-personnalises.md), [export](zone-2-reference/export.md), [historiques](zone-2-reference/historiques-et-inventaires-retardes.md), [import](zone-2-reference/import.md), [inventaires](zone-2-reference/inventaires-sources-et-flux.md), [Meilisearch](zone-2-reference/meilisearch.md), [observers](zone-2-reference/observers-lifecycle-hooks.md), [Connect](zone-2-reference/traitement-connect.md), [inventaires traitement](zone-2-reference/traitement-inventaires.md), [MQTT](zone-2-reference/traitement-mqtt.md).

### [Zone 3 — Exploitation](zone-3-exploitation/infrastructure.md)

Infrastructure, observabilité (Horizon, Nightwatch, SonarQube) et runbooks opérationnels (VPN, BDD, Kanister, MQTT, déploiement).

### [Zone 4 — Annexes](zone-4-annexes/hardware/abeeway.md)

Hardware (Abeeway, downlinks) et archives.

## Maintenance

Le [tableau de bord maintenance](index.md) liste toutes les pages avec leur statut, les pages à revoir, les pages dépréciées et les pages sans owner assigné.

## Conventions

Chaque page porte un front-matter avec `status` (`draft`, `current`, `needs-update`, `deprecated`), `owner`, `last_reviewed` et `review_every`. Voir le [CLAUDE.md](../CLAUDE.md) du repo pour les règles complètes.
