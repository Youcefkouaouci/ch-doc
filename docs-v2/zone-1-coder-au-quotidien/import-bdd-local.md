---
title: Import BDD client en local
status: current
owner: "<À COMPLÉTER>"
last_reviewed: 2026-09-08
review_every: 6m
tags: [dev, bdd, mysql, mongodb, onboarding]
---

# Import BDD client en local (demoV3)

Procédure pour importer une base de données client (MySQL + MongoDB) sur l'environnement local demoV3.

## Étapes

### 1. Créer la base de données MySQL sur demoV3

```bash
sudo create-db nom_de_la_db
```

### 2. Exporter et importer MySQL

Se connecter à la base MySQL du client, l'exporter, puis l'importer sur demoV3.

### 3. Exporter MongoDB du client

Récupérer l'URI de la DB Mongo du client, puis remplacer ce qu'il y a après le `/` par `?authSource=admin&replicaSet=replicaset&tls=true`.

```bash
mongodump --uri "URI_client_modifiée" --db nom_de_la_db_mongo_du_client --out ./backup_nom_du_client
```

### 4. Importer MongoDB sur demoV3

```bash
mongorestore --uri "URI_de_la_db_mongo_demov3" ./backup_nom_du_client
```

### 5. Configurer l'environnement

Modifier le `.env` de demoV3 :

```dotenv
DB_DATABASE=nom_de_la_db_mysql
MDB_DATABASE=nom_de_la_db_mongodb
```

Puis redémarrer :

```bash
./dockerdo stop
./dockerdo start
```

### 6. Anonymiser les utilisateurs (RGPD)

> **Obligatoire** après chaque import de BDD client.

```sql
UPDATE users
SET email = CONCAT('<prenom.nom>+', id, '@charlie-solutions.com'),
    firebaseToken = ''
WHERE email NOT IN (
    'administrateur-charlie@charlie-solutions.com',
    'technical@charlie-solutions.com'
);
```

### 7. Appliquer les migrations et seeders

Se positionner sur la branche `dev`, puis lancer les commandes du sprint en cours :

```bash
./dockerdo art migrate
./dockerdo runSeeder
./dockerdo art meili:setup
```
