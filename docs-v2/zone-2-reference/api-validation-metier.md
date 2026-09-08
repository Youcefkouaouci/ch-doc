---
title: API — Règles de validation métier
status: needs-update
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [api, validation, store, update]
---

{% hint style="warning" %}
**Page en cours de vérification** — les sections Zone et Site sont marquées « à MAJ » et doivent être confrontées au code.
{% endhint %}

# API — Règles de validation métier

Règles de validation par entité et workflows store/update pour les entités principales.

> Les sections Zone et Site sont marquées **à mettre à jour**.
> Source code des validations : `code-charlie/app/Http/Requests/`

## Règles de validation par entité

| Entité | Contraintes |
|--------|------------|
| **Brands** | Nom unique par modèle (matériel, zone, etc.). Au moins un `isFor*` à `true` |
| **Formats** | Nom unique par modèle. Au moins un `isForSensor` ou `isForTracker` à `true` |
| **Materials** | Référence interne unique par **client** (toutes filiales confondues — `UniqueInternalReference`). StatusCode doit appartenir au modèle matériels |
| **Groups** | Nom unique par filiale |
| **Periodicities** | Valeur + unité unique par filiale |
| **QrCodes** | Code unique |
| **Sensors** | Numéro unique. StatusCode valide pour le modèle capteur. `isForZone` ou `isForMaterial` |
| **Sites** | Date de début < date de fin |
| **States** | Au moins un `isFor*` à `true`. Couleur hexadécimale `#XXXXXX` |
| **Status** | Code unique par modèle. Couleur hexadécimale `#XXXXXX` |
| **Teams** | Nom unique par filiale |
| **Trackers** | Numéro unique. StatusCode valide pour le modèle Tracker |
| **Types** | Au moins un `isFor*` à `true`. Nom unique par filiale |
| **Users** | Adresse mail unique |
| **Zones** | Impossible d'associer capteur ET tracker. Tracker non associé à une autre zone. Capteur non associé à une autre zone/matériel |

## Workflows store/update

### Matériel

#### Store

1. Créer le matériel
2. Si localisation (adresse OU lat+lon) → créer `Location` : `address`, `latitude`, `longitude`, `materialId`
3. Si `lastControls` → itérer et créer les `Control` : `object`, `objectId`, `controlFields` (vide), `controlSheetId`, `date`, `isDone=true`
   - Si alerte à contrôler/quarantaine → mettre à jour `conformityId`
4. Si EAV → créer les `Eav` : `object`, `objectId`, `eavs[]`, `subsidiaryId`

Retour API : matériel + localisation + EAV

#### Update

1. Mettre à jour le matériel (champs modifiés uniquement)
2. Location : créer/mettre à jour ou **supprimer** si plus de localisation
3. Controls : idem store
4. EAV : idem store

Retour API : matériel + localisation + EAV

### Zone (à MAJ)

#### Store

1. Créer la zone
2. Si localisation → créer `Location`
3. Si `lastControlDate` → créer `Control`
4. Si EAV → créer les `Eav`

Retour API : zone uniquement

#### Update

1. Mettre à jour la zone
2. Location : créer/mettre à jour ou **supprimer** si plus de localisation
3. EAV : idem store

Retour API : zone uniquement

### Site (à MAJ)

#### Store / Update

Même logique que Zone (Location + Control + EAV).

Retour API : site uniquement

## Notes

- **Contrôles** : les dates de contrôle ne sont pas dans le formulaire mobile actuellement
- **EAV** : pas encore implémenté côté mobile
- Voir aussi : [eav-champs-personnalises.md](eav-champs-personnalises.md), [controles-fiches-templates.md](controles-fiches-templates.md)
