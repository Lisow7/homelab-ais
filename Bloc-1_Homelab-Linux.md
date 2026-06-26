# 🐧 Homelab Linux — Serveur Ubuntu administré en ligne de commande

> Projet réalisé dans le cadre de ma préparation au **Titre Professionnel Administrateur d'Infrastructures Sécurisées (AIS)** — RNCP 37680.
> Outil : **Ubuntu Server 24.04 LTS** dans **VirtualBox** · Auteur : **Lisow**

---

## 🎯 Objectif

Monter **de A à Z** un serveur Ubuntu Server dans une machine virtuelle, et l'**administrer entièrement en ligne de commande** : entretien système, gestion des utilisateurs et des droits, supervision des processus et des services, et automatisation par script Bash.

> 💬 **Ma phrase d'entretien :** « J'administre un serveur Ubuntu Server (monté de A à Z dans VirtualBox) en ligne de commande : mises à jour système, utilisateurs, groupes, permissions, processus, services — et j'automatise des tâches avec des scripts Bash. »

---

## 🧱 Infrastructure de référence

| Élément | Valeur |
|---|---|
| OS | Ubuntu Server 24.04 LTS (*Noble*) |
| Hôte | VirtualBox 7.2.x — Windows / AMD Ryzen 5 5600X, 16 Go |
| Ressources VM | 4096 Mo RAM · 2 CPU · disque 25 Go (LVM) |
| Hostname | `serveur-ais` |
| Utilisateur | `lisow` (+ `paul` créé pour les tests) · Groupe : `equipe-dev` |
| SSH | OpenSSH installé, service **enabled** (port 22) |

---

## 🛠️ Compétences mises en œuvre

| Domaine | Commandes clés |
|---|---|
| **Entretien système** | `sudo apt update` · `sudo apt upgrade` · `sudo apt autoremove` |
| **Utilisateurs & groupes** | `adduser` · `groupadd` · `usermod -aG` · `id` · `getent` |
| **Permissions** | lecture de `ls -l` · `chmod` (770, `+x`) · `chown proprio:groupe` |
| **Navigation & fichiers** | `pwd` · `ls -l` · `cd` · `mkdir` · `touch` · `cp` · `mv` · `rm` · `nano` · `cat` |
| **Processus** | `ps aux` · `top` · `kill` |
| **Services** | `systemctl status / start / stop / enable` |
| **Scripting Bash** | script `sauvegarde.sh` (shebang, `chmod +x`, exécution `./`) |

---

## ✅ Exercice de validation — un dossier partagé d'équipe

Création d'un dossier accessible en écriture au groupe `equipe-dev`, et interdit aux autres :

```bash
sudo groupadd equipe-dev
sudo usermod -aG equipe-dev paul
sudo mkdir /srv/partage-dev
sudo chown root:equipe-dev /srv/partage-dev   # propriétaire : groupe equipe-dev
sudo chmod 770 /srv/partage-dev               # groupe rwx, autres rien
```

**Preuve du bon fonctionnement** — test en tant qu'un membre du groupe :

```bash
su - paul                          # devenir paul
touch /srv/partage-dev/test.txt    # écriture réussie (silence = OK)
exit
```

> 🔎 L'écriture réussie de `paul` dans le dossier prouve que la chaîne **groupe → chown → chmod** est correctement configurée.

---

## 🧠 Compétences démontrées (lien RNCP 37680 — BC01)

- Installation et entretien d'un système Linux serveur.
- Gestion des comptes, des groupes et du modèle de permissions.
- Supervision des processus et des services (`systemctl`).
- Automatisation de tâches d'administration par script Bash.

---

## 📂 Contenu

| Fichier | Description |
|---|---|
| `Aide-memoire-AIS.md` | Mémo des commandes (Bloc 1 Linux + Bloc 2 Réseaux) |
| `README.md` | Présentation générale du homelab |

---

> 🔜 **Pour aller plus loin :** passer la VM en réseau **Pont**, puis s'y connecter en **SSH par clé depuis Windows** (sécurisation — Bloc 3).
