---
title: Champs personnalisés (EAV)
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [eav, champs-personnalisés, customfields]
---

# Champs personnalisés (EAV)

Système Entity-Attribute-Value pour ajouter des champs personnalisés aux formulaires.

- **CustomFields** = définition du champ (label, type, section, portée)
- **EAV** = valeur remplie dans un champ personnalisé

## Structure de données

```
customFields {
  id, attribute, type, isRequired, section,
  object, objectId, subObject,
  -- subsidiaries via pivot table
  fromObject, fromObjectId
}

eav {
  id, customFieldId, objectId, value
}
```

### Définition des clés

| Clé | Rôle |
|-----|------|
| `attribute` | Nom/label du champ |
| `type` | Type de valeur : `string`, `int`, `date`, `datetime-local` |
| `isRequired` | Champ obligatoire dans le formulaire |
| `section` | Section du formulaire (1–5, voir ci-dessous) |
| `subsidiaries` | Filiales (relation many-to-many via table pivot `custom_fields_subsidiaries`) |
| `object` | Type d'asset lié (`App\Models\Type`, `Material`, `Zone`, etc.) |
| `objectId` | ID de l'asset lié (null = tous les assets de ce type) |
| `subObject` | Sous-type, seulement si `object = App\Models\Type` |
| `fromObject` | Asset parent de l'objet |
| `fromObjectId` | ID du parent |

### Sections

| N° | Section | Note |
|----|---------|------|
| 1 | Informations générales | |
| 2 | Affectation | Apparaît dans infos générales sur le détail, dans Affectation en create/edit. Globalement non utilisée |
| 3 | Connexion (`SECTION_CONNECTION`) | |
| 4 | État et conformité | |
| 5 | Informations complémentaires | Globalement non utilisée |

## API — Exemples de payloads

### Créer un CustomField

```json
{
  "subsidiaries": [3],
  "attribute": "Datetime",
  "type": "datetime-local",
  "section": 1,
  "object": "App\\Models\\Material",
  "objectId": 4
}
```

> **Attention** : `subsidiaries` est un **tableau** d'IDs (relation many-to-many), pas un scalaire `subsidiaryId`.

### Créer un EAV

```json
{
  "customFieldId": 10,
  "objectId": 11,
  "value": "2025-10-01 15:22",
  "timezone": "Europe/Paris"
}
```

> `timezone` est nécessaire uniquement pour le type `datetime-local`.

## 6 cas d'usage

| Cas | object | objectId | subObject | fromObject | fromObjectId | Portée |
|-----|--------|----------|-----------|------------|-------------|--------|
| 1 | `Type` | ID | null | null | null | Un type précis |
| 2 | `Type` | null | `Zone` | null | null | Tous les types d'une catégorie |
| 3 | `Type` | null | null | null | null | Tous les types |
| 4 | `Material` | null | null | `Type` | ID | Tous les matériels d'un type |
| 5 | `Material` | null | null | null | null | Tous les matériels |
| 6 | `Material` | ID | null | null | null | Un matériel précis |
