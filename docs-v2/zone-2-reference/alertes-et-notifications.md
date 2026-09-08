---
title: Alertes et notifications
status: needs-update
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [alertes, notifications, push, firebase, cron]
---

{% hint style="warning" %}
**Page en cours de vérification** — questions ouvertes : règles de conflit non implémentées (voir `[CODE?]` §Règles de décision) et payloads push enrichis non implémentés (voir `[CODE?]` §Payload push enrichi).
{% endhint %}

# Alertes et notifications

Système d'alertes de la plateforme : types, paramètres, payloads JSON, règles de conflit et envoi de notifications.

> Codes d'alertes : voir [codes-de-statut.md](codes-de-statut.md#alertes--code)

## Types d'alertes

| Code | Constante | Déclencheur | Effet | Schedule |
|------|-----------|-------------|-------|----------|
| 1 | `TO_CONTROL` | X jours avant la date de prochain contrôle | Conformité → « À contrôler » | Tous les jours à 8h (mail récap) |
| 2 | `QUARANTINE` | Date de prochain contrôle dépassée | Conformité → « En quarantaine » | Tous les jours à 8h |
| 3 | `LOST` | Pas de détection depuis X jours | Statut → « Perdu » | Tous les jours à 8h |
| 4 | `MOTIONLESS` | Position inchangée depuis X jours (seuil par défaut : 50 m, configurable via `distanceForMovement`, min 50 m) | Statut → « Immobile » | Tous les jours à 8h |
| 5 | `GEOFENCING` | Détection dans/hors d'un site | Association/désassociation au site | Dans les traitements (temps réel) |

**Alerte Retrouvé** : se déclenche quand un élément « Perdu » est à nouveau détecté. Le statut « Retrouvé » reste 3 jours. Notifications envoyées aux utilisateurs configurés sur l'alerte Perdu. Déclenché dans les traitements.

### Liens Figma (workflows détaillés)

- [Alerte Immobile V3](https://www.figma.com/board/5621nbJN2vOQTqboyb5Gik/Alerte-immobile-V3)
- [Alerte À Contrôler / En Quarantaine V3](https://www.figma.com/board/2pfa9nExMhGE2ZDDNf7T4B/Alerte-à-Contrôler-%2F-en-quarantaine-V3)
- [Alerte Perdu V3](https://www.figma.com/board/q2pcktlrM8y6rW9jVlKh1Y/Alerte-Perdu-(V3))
- [Alerte Retrouvé](https://www.figma.com/board/XHwIYVpR3WtOkZ7kDQ4M6z/Alerte-Retrouvé)
- [Alerte Geofencing V3](https://www.figma.com/board/jsk8bRNxAV9BVr0suRnKJ8/Alerte-geofencing-V3)

## Structure JSON de la table `alerts`

```json
{
  "name": "Example",
  "description": "This is an example description",
  "code": 1,
  "object": "App\\Models\\Zone",
  "objectIds": [1, 2, 3],
  "daysbeforeXXX": 10,
  "isActive": true,
  "notify": {
    "App\\Models\\User": [
      {
        "id": 1,
        "notifications": [
          { "days": 7, "method": [] },
          { "days": 14, "method": ["email"] }
        ]
      }
    ],
    "App\\Models\\Team": []
  }
}
```

### Champs obligatoires

| Champ | Rôle |
|-------|------|
| `code` | Type de l'alerte (1–5) |
| `object` | Type d'objet (`App\Models\Zone`, `Material`, etc.) |
| `objectIds` | Liste des IDs d'objets associés |
| `notify` | Qui notifier (utilisateurs et/ou équipes), avec niveaux de jours et méthodes |

### Champs facultatifs

| Champ | Rôle |
|-------|------|
| `description` | Description libre |

### Paramètres par type d'alerte

| Type | Paramètres spécifiques |
|------|----------------------|
| À contrôler (1) | `daysBeforeControl` — jours avant la date de contrôle. `notify` contient `toControl` (X jours AVANT) et `quarantine` (X jours APRÈS) |
| Immobile (4) | `daysWithoutMovements` — jours sans bouger. `distanceForMovement` — périmètre en mètres (minimum 50 m, défaut 50 m — `Alert::DEFAULT_DISTANCE_FOR_MOVEMENT`) |
| Perdu (3) | `daysWithoutDetections` — jours sans détection |
| Geofencing (5) | `sitesGeofenced` — liste de sites avec `in`/`out`/`isRecurring` par site |

### Payload geofencing `sitesGeofenced`

```json
{
  "sitesGeofenced": {
    "1": { "in": true, "out": false, "isRecurring": false },
    "2": { "in": true, "out": true, "isRecurring": true }
  }
}
```

## Règles de décision sur les conflits

> **`[CODE?]` Non implémentées** — ces règles sont des décisions produit documentées mais **aucune logique de priorité n'existe dans le code**. Les conflits sont actuellement bloqués à la création (`alertAlreadyExistsForObjectId`) plutôt que résolus à la notification. À trancher avec l'équipe.

1. **Plus restreint en priorité** : matériel > type de matériel ; utilisateur > équipe
2. **Plusieurs durées** : prendre la durée la plus grande pour changer l'état/statut
3. **Jours alerte vs jours notification** :
   - À CONTROLER : jours pour la MAJ de l'état ≤ au plus petit niveau de notification
   - IMMOBILE : jours pour la MAJ de l'état ≥ au plus petit niveau de notification
4. **Regrouper les notifications par trame** pour éviter le spam (ex : 5 matériels → 1 notification au lieu de 5)

## Envoi de notifications

> Ancienne page « Crons » — le contenu concerne les notifications push et locales, pas les tâches planifiées.

### Notifications push (Firebase)

Méthode : `sendPushNotification` dans `NotificationManager`

Paramètres : titre, corps (contenu), jeton Firebase (`deviceToken`), identifiant d'objet, type de notification.

Envoyées par les alertes **Geofencing** et **Retrouvé** (via leurs propres jobs `app/Jobs/Alerts/Geofencing.php` et `app/Jobs/Alerts/Found.php`), ainsi que par le récap journalier (`app/Jobs/Alerts/MailDaily.php`). Tous appellent la helper `sendNotifications` (`app/Helpers/alerts.php`).

#### Payload push actuel (implémenté)

Le payload `data` envoyé par `sendPushNotification` est actuellement minimal :

```json
{ "id": "<objectId>", "type": null }
```

> `objectId` et `type` ne sont pas transmis par les call sites actuels — ils prennent leurs valeurs par défaut (`1` et `null`).

#### Payload push enrichi (proposé, non implémenté)

> **`[CODE?]`** Les payloads enrichis ci-dessous sont des **décisions de design non encore implémentées**. À traiter comme les colonnes `data`/`type` de la table notifications.

**Geofencing** : `data.inputMaterialIds`, `data.outputMaterialIds`, `data.inputZoneIds`, `data.outputZoneIds`, `data.type = "geofencing"`.

**Retrouvé** : `data.materialIds`, `data.zoneIds`, `data.type = "found"`.

**Récap journalier** : `data.type = "dailyRecap"`.

Redirection envisagée : un seul asset → détail ; plusieurs → liste.

### Notifications locales (table BDD)

Créées via la helper `sendNotifications` (`app/Helpers/alerts.php`), appelée depuis `MailDaily`, `Found` et `Geofencing`.

| Champ | Rôle |
|-------|------|
| `userId` | Utilisateur notifié |
| `title` | Titre (traduit) |
| `content` | Contenu dynamique (traduit) |
| `url` | Lien de redirection éventuel |

### Décisions de design (notifications)

- Nouveau champ `data` (JSON) dans la table notifications pour les IDs d'assets et liens
- Nouveau champ optionnel `type`
- Côté web : 2 boutons max, site de détection pour distinguer
- Côté mobile : slider pour distinguer matériels/zones
