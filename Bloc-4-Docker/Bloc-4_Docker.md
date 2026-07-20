# 🐳 Docker — Conteneurisation d'une application web

> Projet réalisé dans le cadre de ma préparation au **Titre Professionnel Administrateur d'Infrastructures Sécurisées (AIS)** — RNCP 37680.
> Outil : **Docker Engine** sur Ubuntu Server 24.04 LTS (VM VirtualBox) · Auteur : **Lisow**

---

## 🎯 Objectif

Installer et maîtriser Docker pour construire, déployer et orchestrer des applications conteneurisées : de l'installation du moteur à une stack définie en `docker-compose`, en passant par la construction d'une image personnalisée et la persistance des données.

> 💬 **Ma phrase d'entretien :** « Je conteneurise des applications avec Docker : installation du moteur via le dépôt officiel, écriture de Dockerfile pour construire mes propres images, gestion des volumes pour la persistance des données, et orchestration avec docker-compose. »

---

## 🧱 Contexte

| Élément | Valeur |
|---|---|
| Serveur | Ubuntu Server 24.04 LTS — VM VirtualBox `serveur-ais` |
| Docker Engine | 5:29.6.2 (dépôt officiel APT `download.docker.com`) |
| Composants | `docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin`, `docker-compose-plugin` |

---

## 🔧 Réalisation

### 1. Installation de Docker Engine (dépôt officiel)
```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Validation : `sudo systemctl status docker` → `active (running)` ; `docker run hello-world` → message de confirmation officiel.

**Utilisation sans `sudo` :**
```bash
sudo usermod -aG docker lisow
```
(déconnexion/reconnexion nécessaire pour appliquer le nouveau groupe)

### 2. Premier conteneur avec une image officielle
```bash
docker run -d --name mon-nginx -p 8080:80 nginx
```
`-d` = arrière-plan · `-p 8080:80` = mappe le port 8080 de l'hôte vers le port 80 du conteneur.
Validation : `docker ps` (statut `Up`) puis accès navigateur `http://<IP-VM>:8080` → page d'accueil Nginx.

### 3. Construction d'une image personnalisée (Dockerfile)
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```
```bash
docker build -t mon-site:v1 .
docker run -d --name mon-site-conteneur -p 8081:80 mon-site:v1
```
Validation : `docker images` (image `mon-site:v1` listée) ; accès navigateur `http://<IP-VM>:8081` → page personnalisée servie.

### 4. Persistance des données (volumes)
Démonstration du problème : une modification faite **à l'intérieur** d'un conteneur disparaît après `docker rm` + relance — un conteneur est jetable par nature.

Solution avec un volume (liaison dossier hôte ↔ dossier conteneur) :
```bash
docker run -d --name mon-site-conteneur -p 8081:80 \
  -v ~/mon-site-docker/contenu-web:/usr/share/nginx/html mon-site:v1
```
Validation : une modification faite **depuis la VM** (`echo ... >> contenu-web/index.html`) apparaît instantanément dans le navigateur, sans reconstruire l'image ni relancer le conteneur.

### 5. Orchestration avec docker-compose
```yaml
services:
  site-web:
    image: mon-site:v1
    ports:
      - "8081:80"
    volumes:
      - ./contenu-web:/usr/share/nginx/html
```
```bash
docker compose up -d
docker compose ps
```
Compose crée automatiquement un réseau dédié au projet et nomme le conteneur `<dossier>-<service>-1`. Toute la stack se relance avec une seule commande, à partir d'un fichier versionnable.

---

## ✅ Tests & preuves

| Test | Commande / action | Résultat |
|---|---|---|
| Moteur actif | `sudo systemctl status docker` | `active (running)` |
| Conteneur officiel | `docker run hello-world` | Message de confirmation Docker |
| Image personnalisée servie | Navigateur → port 8081 | Page HTML personnalisée affichée |
| Volume — perte sans volume | Modif interne → `docker rm` → relance | Modification perdue (comportement attendu) |
| Volume — persistance | Modif depuis la VM (hôte) | Modification visible immédiatement dans le conteneur |
| Compose | `docker compose ps` | Service `site-web` `Up`, port `8081` actif |

---

## 🧠 Compétences démontrées (lien RNCP 37680 — BC01)

- Installation d'un moteur de conteneurisation depuis un dépôt officiel tiers.
- Construction d'images personnalisées via Dockerfile (`FROM`, `COPY`).
- Gestion du cycle de vie des conteneurs (`run`, `stop`, `rm`, `exec`, `logs`).
- Compréhension et mise en œuvre de la persistance des données (volumes).
- Orchestration multi-conteneurs déclarative avec docker-compose.
- Exposition de services via le mapping de ports.

---

## 🐞 Mémo des erreurs vécues (Docker)

| Message / symptôme | Cause | Correction |
|---|---|---|
| `apt install -m 0755 -d ...` échoue | Confusion entre `apt install` (paquets) et `install` (créer fichier/dossier) | `sudo install -m 0755 -d /etc/apt/keyrings` |
| `Malformed entry ... (Component)` sur `apt update` | La substitution `$VERSION_CODENAME` a échoué au collage, ligne de dépôt incomplète | Réécrire la ligne avec le codename en dur (`noble`) |
| `Unable to locate package doker-buildx-plugin` | Faute de frappe dans le nom du paquet | Utiliser `Tab` pour autocompléter les noms de paquets |
| Modification perdue après `docker rm` + relance | Écriture faite **dans** le conteneur, sans volume | Toujours utiliser `-v` pour toute donnée qui doit survivre au conteneur |
| Presse-papier ne fonctionne pas dans la console VirtualBox | Guest Additions non installées | Utiliser une session **SSH** (PowerShell) à la place pour le copier-coller |

---

## 📂 Contenu du dépôt

| Fichier | Description |
|---|---|
| `Dockerfile` | Recette de construction de l'image `mon-site:v1` |
| `docker-compose.yml` | Orchestration du service `site-web` (port + volume) |
| `index.html` | Page web personnalisée servie par le conteneur |
| `README.md` | Ce document |

---

> **Bloc 4 — Docker : terminé.** Prochaine étape : suite du programme des modules AIS.
