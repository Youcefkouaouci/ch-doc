---
title: Index de la documentation
status: current
owner: "<à assigner>"
last_reviewed: 2026-09-08
review_every: 3m
tags: [index, navigation]
---

# Index de la documentation — Charlie Platform V3

## Table des pages

### Zone 1 — Coder au quotidien

| Page | Status | Owner | Dernière revue |
|------|--------|-------|----------------|
| [Accès externes](zone-1-coder-au-quotidien/acces-externes.md) | current | <à assigner> | 2026-09-08 |
| [Commandes utiles](zone-1-coder-au-quotidien/commandes-utiles.md) | current | <à assigner> | 2026-09-08 |
| [Import BDD client en local](zone-1-coder-au-quotidien/import-bdd-local.md) | current | <à assigner> | 2026-09-08 |
| [Seeders utiles et gestion des rôles](zone-1-coder-au-quotidien/seeders-et-roles.md) | current | <à assigner> | 2026-09-08 |

### Zone 2 — Référence plateforme

| Page | Status | Owner | Dernière revue |
|------|--------|-------|----------------|
| [Alertes et notifications](zone-2-reference/alertes-et-notifications.md) | **needs-update** | <à assigner> | 2026-09-08 |
| [API — Règles de validation métier](zone-2-reference/api-validation-metier.md) | **needs-update** | <à assigner> | 2026-09-08 |
| [Codes de statut et d'état](zone-2-reference/codes-de-statut.md) | current | <à assigner> | 2026-09-08 |
| [Configuration GPS et anonymisation RGPD](zone-2-reference/configuration-gps-anonymisation.md) | current | <à assigner> | 2026-09-08 |
| [Contrôles — Fiches et templates](zone-2-reference/controles-fiches-templates.md) | current | <à assigner> | 2026-09-08 |
| [Champs personnalisés (EAV)](zone-2-reference/eav-champs-personnalises.md) | current | <à assigner> | 2026-09-08 |
| [Export](zone-2-reference/export.md) | current | <à assigner> | 2026-09-08 |
| [Historiques et inventaires retardés](zone-2-reference/historiques-et-inventaires-retardes.md) | current | <à assigner> | 2026-09-08 |
| [Import — Valeurs acceptées](zone-2-reference/import.md) | current | <à assigner> | 2026-09-08 |
| [Inventaires : sources, codes et flux](zone-2-reference/inventaires-sources-et-flux.md) | current | <à assigner> | 2026-09-08 |
| [Meilisearch](zone-2-reference/meilisearch.md) | current | <à assigner> | 2026-09-08 |
| [Observers et hooks de cycle de vie](zone-2-reference/observers-lifecycle-hooks.md) | **needs-update** | <à assigner> | 2026-09-08 |
| [Traitement Charlie Connect](zone-2-reference/traitement-connect.md) | current | <à assigner> | 2026-09-08 |
| [Traitement des inventaires](zone-2-reference/traitement-inventaires.md) | current | <à assigner> | 2026-09-08 |
| [Traitement MQTT](zone-2-reference/traitement-mqtt.md) | current | <à assigner> | 2026-09-08 |

### Zone 3 — Exploitation

| Page | Status | Owner | Dernière revue |
|------|--------|-------|----------------|
| [Infrastructure](zone-3-exploitation/infrastructure.md) | draft | <à assigner> | 2026-09-08 |
| [Procédure de déploiement](zone-3-exploitation/procedure-deploiement.md) | current | <à assigner> | 2026-09-08 |
| [Laravel Horizon](zone-3-exploitation/observabilite/laravel-horizon.md) | current | <à assigner> | 2026-09-08 |
| [Laravel Nightwatch](zone-3-exploitation/observabilite/laravel-nightwatch.md) | current | <à assigner> | 2026-09-08 |
| [SonarQube](zone-3-exploitation/observabilite/sonarqube.md) | current | <à assigner> | 2026-09-08 |
| [Serveur MQTT](zone-3-exploitation/runbooks/serveur-mqtt.md) | **needs-update** | <à assigner> | 2026-09-08 |
| [VPN — Installation et utilisation](zone-3-exploitation/runbooks/vpn.md) | draft | <à assigner> | 2026-09-08 |
| [Gestion des BDD et création de client](zone-3-exploitation/runbooks/gestion-bdd-et-client.md) | draft | <à assigner> | 2026-09-08 |
| [Procédure — Relancer Kanister](zone-3-exploitation/runbooks/relancer-kanister.md) | draft | <à assigner> | 2026-09-08 |

### Zone 4 — Annexes

| Page | Status | Owner | Dernière revue |
|------|--------|-------|----------------|
| [Abeeway — Tracker GPS](zone-4-annexes/hardware/abeeway.md) | draft | <à assigner> | 2026-09-08 |
| [Downlink V3](zone-4-annexes/hardware/downlink-v3.md) | draft | <à assigner> | 2026-09-08 |

---

## À revoir

Pages en `needs-update` ou dont `last_reviewed` + `review_every` est dépassé.

| Page | Status | Raison |
|------|--------|--------|
| [Alertes et notifications](zone-2-reference/alertes-et-notifications.md) | needs-update | `[CODE?]` règles de conflit + payloads enrichis non implémentés |
| [API — Règles de validation métier](zone-2-reference/api-validation-metier.md) | needs-update | Sections Zones/Sites « à MAJ » |
| [Observers et hooks de cycle de vie](zone-2-reference/observers-lifecycle-hooks.md) | needs-update | `[CODE?]` SimplicitiService non documentée |
| [Serveur MQTT](zone-3-exploitation/runbooks/serveur-mqtt.md) | needs-update | Contenu à mettre à jour |

> Aucune page n'a dépassé son `review_every` au 2026-09-08 (toutes revues à cette date).

---

## Dépréciées

Aucune page dépréciée actuellement publiée.

---

## Sans owner

**Toutes les pages (30)** ont `owner: "<à assigner>"`. Aucun owner réel n'a été assigné.

| Zone | Pages sans owner |
|------|-----------------|
| Zone 1 | acces-externes, commandes-utiles, import-bdd-local, seeders-et-roles |
| Zone 2 | alertes-et-notifications, api-validation-metier, codes-de-statut, configuration-gps-anonymisation, controles-fiches-templates, eav-champs-personnalises, export, historiques-et-inventaires-retardes, import, inventaires-sources-et-flux, meilisearch, observers-lifecycle-hooks, traitement-connect, traitement-inventaires, traitement-mqtt |
| Zone 3 | infrastructure, procedure-deploiement, laravel-horizon, laravel-nightwatch, sonarqube, serveur-mqtt, vpn, gestion-bdd-et-client, relancer-kanister |
| Zone 4 | abeeway, downlink-v3 |
