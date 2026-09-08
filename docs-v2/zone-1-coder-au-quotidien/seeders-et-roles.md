---
title: Seeders utiles et gestion des rôles
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [dev, seeders, rôles, permissions]
---

# Seeders utiles et gestion des rôles

Certains seeders sont utiles pour débuguer une fonctionnalité complexe rapidement. Ils peuvent être instanciés dans le seeder par défaut (`DatabaseSeeder`) sans être committés sur le dépôt.

## Génération du mail journalier

Seeder pour générer le template du mail journalier sans avoir à créer manuellement les alertes et conditions :

```php
$user = User::first();

if (! $user) {
    $this->command->warn('No user found.');
    return;
}

$data = [
    Alert::TO_CONTROL_CODE => [
        [
            'object' => 'App\Models\Material',
            'days'   => 0,
        ],
    ],
    Alert::QUARANTINE_CODE => [
        [
            'object' => 'App\Models\Material',
            'days'   => 4,
        ],
    ],
    Alert::MOTIONLESS_CODE => [
        [
            'object' => 'App\Models\Material',
            'days'   => 2,
        ],
    ],
];

Mail::to($user->email)->send(new DailyMail($user, $data));

$this->command->info("DailyMail sent to {$user->email}");
```

> Les codes d'alertes (`TO_CONTROL_CODE`, `QUARANTINE_CODE`, `MOTIONLESS_CODE`) sont définis dans le modèle `Alert`. Voir [codes-de-statut.md](../zone-2-reference/codes-de-statut.md).

## Suppression en masse de capteurs

Placer le fichier `sensors_to_delete.csv` (contenant les numéros de capteurs) dans `tmp/` sur S3, puis exécuter le seeder dédié.

## Création / modification de rôles partenaires

Procédure interactive via le seeder `CreatePartnersRoleSeeder`.

### Créer un nouveau rôle

1. Se connecter au worker plateforme du client
2. Lancer :

```bash
php artisan db:seed --class=CreatePartnersRoleSeeder
```

3. Choisir `[0]` : Ajouter un rôle
4. À la question « est-ce que c'est un partenaire ? » → répondre **OUI**
5. Sélectionner les endpoints (touche **espace** pour cocher)

### Modifier un rôle existant

1. Se connecter au worker plateforme du client
2. Lancer :

```bash
php artisan db:seed --class=CreatePartnersRoleSeeder
```

3. Choisir `[1]` : Modifier un rôle, sélectionner le rôle
4. Assigner les nouveaux endpoints
5. Après validation, vider le cache des permissions :

```bash
php artisan permission:cache-reset
```
