# Mission : refonte de la documentation technique (Charlie Platform V3)

## Sources
- `code-charlie/`  : code source = VÉRITÉ du "comment ça marche". Ne pas le modifier.
- `notion-doc/`    : export Notion = doc actuelle écrite à la main, à affiner.
- `docs-v2/`       : CIBLE. Toute la doc refondue s'écrit ici. Ne jamais écraser notion-doc.

## Règle générale de documentation (impérative)
On documente UNIQUEMENT :
- les fonctionnalités complexes (avec un workflow découpé, étape par étape) ;
- les codes d'état / traitements / liaisons à un asset ;
- les payloads (JSON/YAML...) avec le rôle de CHAQUE champ.
Sinon, le CODE sert de doc : on lie au fichier source (chemin + classe/méthode) au lieu de
recopier ou reformuler ce qu'il fait déjà. Objectif permanent : ZÉRO duplication.

## Structure cible (axe unique : le but)
- Zone 1 · Coder au quotidien : démarrer (accès, DB locale, commandes, seeders), + liens
  Architecture/Conventions (dans le repo) + ADR (décisions).
- Zone 2 · Référence "comment marche la plateforme" : suit le découpage du CODE
  (HTTP/Contrôleurs, Form Requests, Domain Managers, EAV, Audit, IoT/Ingestion,
  Contrôles, Frontend Inertia/Vue, OpenAPI, Recherche, Tests/Ops). C'est le gros.
- Zone 3 · Exploitation : Infrastructure, Observabilité (Sonarqube/Horizon/Nightwatch),
  Runbooks (VPN, gestion BDD, relancer kanister, création client), MAJ/Migrations (process).
- Zone 4 · Annexes : Hardware (Abeeway, downlinks), Archives.

## Séparation des types (ne pas mélanger)
- Référence (comment c'est construit)        → Zone 2
- Runbook (comment j'exécute une action)     → Zone 3 · Runbooks
- Journal (trace de ce qui a été fait)       → N'EST PAS de la doc → à sortir vers Jira
- Décision (pourquoi tel choix)              → ADR (Zone 1)

## Convention de cycle de vie (front-matter de CHAQUE page docs-v2)
```yaml
---
title: <titre clair>
status: current            # draft | current | needs-update | deprecated
owner: "<@personne>"       # une personne, pas "l'équipe"
last_reviewed: <YYYY-MM-DD>
review_every: 6m           # 3m | 6m | 12m
superseded_by:             # si deprecated → lien vers la page qui remplace
tags: [<domaine>]
---
```
Règle d'or : on ne SUPPRIME pas une info obsolète, on la passe en `deprecated` avec
`superseded_by`. Les ex-tags "A MAJ" du titre deviennent `status: needs-update`.

## Interdits (anti-hallucination)
- Ne jamais inventer une valeur interne (URL, version de service, secret, chemin d'infra).
  Si l'info n'est ni dans le code ni dans notion-doc : écrire `<À COMPLÉTER>` + un TODO.
- Ne pas reformuler le code en prose : lier au fichier source.
- Langue : français. Noms techniques en anglais quand c'est l'usage.
