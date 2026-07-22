# 🗄️ SQL — Base de données relationnelle conteneurisée (PostgreSQL)

> Projet réalisé dans le cadre de ma préparation au **Titre Professionnel Administrateur d'Infrastructures Sécurisées (AIS)** — RNCP 37680.
> Outil : **PostgreSQL 16** en conteneur **Docker** sur Ubuntu Server 24.04 LTS (VM VirtualBox) · Auteur : **Lisow**

---

## 🎯 Objectif

Installer et administrer une base de données relationnelle : modéliser des tables liées par clés primaire/étrangère, manipuler les données en SQL (CRUD), garantir l'intégrité référentielle, et gérer des comptes avec des droits d'accès différenciés (principe de moindre privilège).

> 💬 **Ma phrase d'entretien :** « J'ai déployé une base PostgreSQL en conteneur Docker, avec persistance des données par volume. J'ai modélisé un schéma relationnel simple (deux tables liées par clé étrangère), pratiqué le CRUD complet en SQL, et mis en place une gestion des droits d'accès avec un rôle en lecture seule — le principe de moindre privilège appliqué concrètement. »

---

## 🧱 Contexte

| Élément | Valeur |
|---|---|
| Serveur | Ubuntu Server 24.04 LTS — VM VirtualBox `serveur-ais` |
| SGBD | PostgreSQL 16 (image officielle `postgres:16`, conteneur Docker) |
| Conteneur | `pg-ais`, port `5432`, volume `~/pg-ais-data` |
| Choix technique | PostgreSQL retenu plutôt que MySQL : conformité au standard SQL, gouvernance communautaire, cohérent avec le Bloc 4 (conteneurisation) |

---

## 🔧 Réalisation

### 1. Déploiement de PostgreSQL en conteneur Docker
```bash
docker run -d --name pg-ais \
  -e POSTGRES_PASSWORD=motdepasse123 \
  -e POSTGRES_USER=lisow \
  -e POSTGRES_DB=ais_db \
  -p 5432:5432 \
  -v ~/pg-ais-data:/var/lib/postgresql/data \
  postgres:16
```
Le volume (`-v`) garantit la persistance des données : sans lui, tout serait perdu à la suppression du conteneur (principe déjà vu au Bloc 4).

Validation : `docker ps` → conteneur `pg-ais` à l'état `Up`.

### 2. Connexion au client `psql`
```bash
docker exec -it pg-ais psql -U lisow -d ais_db
```

### 3. Modélisation d'un schéma relationnel
Deux tables liées par une clé primaire / clé étrangère :
```sql
CREATE TABLE utilisateurs (
    id SERIAL PRIMARY KEY,
    nom VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE commandes (
    id SERIAL PRIMARY KEY,
    produit VARCHAR(100) NOT NULL,
    user_id INTEGER REFERENCES utilisateurs(id)
);
```
`REFERENCES utilisateurs(id)` impose l'**intégrité référentielle** : impossible d'insérer une commande pointant vers un utilisateur inexistant.

### 4. CRUD complet
```sql
-- Create
INSERT INTO utilisateurs (nom, email) VALUES ('Alice', 'alice@mail.com'), ('Bob', 'bob@mail.com');
INSERT INTO commandes (produit, user_id) VALUES ('Clavier', 1), ('Souris', 1), ('Écran', 2);

-- Read (+ jointure)
SELECT utilisateurs.nom, commandes.produit
FROM commandes
JOIN utilisateurs ON commandes.user_id = utilisateurs.id;

-- Update
UPDATE utilisateurs SET email = 'alice.nouvelle@mail.com' WHERE id = 1;

-- Delete
DELETE FROM commandes WHERE id = 2;
```
> ⚠️ Réflexe systématique : tester le `WHERE` avec un `SELECT` avant tout `UPDATE`/`DELETE`, pour ne jamais modifier/supprimer la mauvaise ligne — ou toutes les lignes en cas d'oubli du `WHERE`.

### 5. Gestion des rôles et droits d'accès
```sql
CREATE ROLE lecteur WITH LOGIN PASSWORD 'lecteur123';
GRANT CONNECT ON DATABASE ais_db TO lecteur;
GRANT SELECT ON utilisateurs, commandes TO lecteur;
```
Le rôle `lecteur` peut lire les données mais pas les modifier — principe de moindre privilège.

---

## ✅ Tests & preuves

| Test | Commande / action | Résultat | Statut |
|---|---|---|---|
| Conteneur actif | `docker ps` | `pg-ais` `Up`, port `5432` exposé | ✅ |
| Tables créées | `\dt` | `utilisateurs` et `commandes` listées | ✅ |
| Intégrité référentielle | `INSERT ... user_id = 99` (inexistant) | `ERROR: violates foreign key constraint` | ✅ |
| Jointure | `SELECT ... JOIN ...` | Nom + produit correctement associés (3 lignes) | ✅ |
| UPDATE ciblé | `UPDATE ... WHERE id = 1` | `UPDATE 1`, seule la ligne visée modifiée | ✅ |
| DELETE ciblé | `DELETE ... WHERE id = 2` | `DELETE 1`, seule la ligne visée supprimée | ✅ |
| Droits en lecture seule | `SELECT` en tant que `lecteur` | Fonctionne | ✅ |
| Droits refusés en écriture | `INSERT` en tant que `lecteur` | `ERROR: permission denied for table utilisateurs` | ✅ |

---

## 🧠 Compétences démontrées (lien RNCP 37680 — support transverse)

- Déploiement d'un SGBD en environnement conteneurisé avec persistance des données.
- Modélisation d'un schéma relationnel (clé primaire, clé étrangère, contraintes).
- Maîtrise du CRUD complet en SQL (`INSERT`, `SELECT`, `UPDATE`, `DELETE`).
- Compréhension et démonstration de l'intégrité référentielle.
- Jointures pour recombiner des données normalisées.
- Gestion des comptes et des droits d'accès à une base (principe de moindre privilège).

---

## 📂 Contenu du dépôt

| Fichier | Description |
|---|---|
| `Aide-memoire-AIS.md` | Mémo des commandes (Blocs 1 à 5 + Git) |
| `README.md` | Ce document |

---

> **Bloc 5 — SQL : terminé.** Prochaine étape : Bloc 6 — Projet final (Fridge+ en ligne, supervisé).
