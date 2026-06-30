# 🔒 Sécurité — Accès distant SSH durci (Ubuntu Server)

> Projet réalisé dans le cadre de ma préparation au **Titre Professionnel Administrateur d'Infrastructures Sécurisées (AIS)** — RNCP 37680.
> Outil : **Ubuntu Server 24.04 LTS** (VM VirtualBox, réseau Pont) · Client : **OpenSSH (Windows)** · Auteur : **Lisow**
> 🚧 Bloc en cours — parties suivantes à venir : pare-feu **UFW**, **Fail2Ban**, **ANSSI/RGPD**.

---

## 🎯 Objectif

Sécuriser l'accès distant à un serveur Linux : passer d'un accès par mot de passe à une **authentification par clé SSH uniquement**, désactiver le compte **root** et **refuser le mot de passe**, dans une démarche alignée sur les bonnes pratiques de l'**ANSSI**.

> 💬 **Ma phrase d'entretien :** « Je sécurise un serveur Linux exposé : accès SSH par clé uniquement (root désactivé, mot de passe refusé). Je sais diagnostiquer la configuration réellement appliquée avec `sshd -T` et résoudre les conflits de fichiers *drop-in* — ce que j'ai justement dû faire pour neutraliser un réglage `cloud-init` qui réactivait l'authentification par mot de passe. »

---

## 🧱 Contexte

| Élément | Valeur |
|---|---|
| Serveur | Ubuntu Server 24.04 LTS — VM VirtualBox `serveur-ais` |
| Réseau | Mode **Pont** (Bridged) → IP `192.168.1.35/24` (DHCP box) |
| Poste d'administration | Windows — client **OpenSSH** natif (PowerShell) |
| Type de clé | **ed25519** (algorithme moderne recommandé) |

---

## 🔧 Étapes de réalisation

### 1. Réseau Pont (prérequis)
Passage de la carte réseau de la VM de **NAT** (réseau privé `10.0.2.x`, injoignable) à **Pont** : la VM reçoit alors une IP du réseau local et devient joignable depuis Windows.
Validation : `ip a` → `inet 192.168.1.35` sur `enp0s3`, puis `ping 192.168.1.35` depuis Windows (0 % de perte, `TTL=64` → même réseau local, aucun routeur traversé).

### 2. Authentification par clé
```powershell
# Côté Windows : génération de la paire (clé privée jamais partagée)
ssh-keygen -t ed25519

# Dépôt de la clé PUBLIQUE sur la VM, droits stricts inclus
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh lisow@192.168.1.35 `
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```
Validation : `ssh lisow@192.168.1.35` se connecte **sans mot de passe**.

### 3. Durcissement du serveur SSH
Configuration via un fichier *drop-in* dédié (sans modifier le fichier d'origine) :
```bash
sudo tee /etc/ssh/sshd_config.d/99-durcissement.conf > /dev/null <<'EOF'
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
EOF

sudo sshd -t                 # test syntaxe AVANT d'appliquer (silence = OK)
sudo systemctl reload ssh    # reload : n'interrompt pas les sessions en cours
```
> Bonne pratique appliquée : **toujours garder une 2ᵉ session SSH ouverte** comme filet de sécurité avant de recharger la config, pour éviter tout auto-verrouillage.

---

## 🐞 Incident rencontré & résolu — conflit de *drop-in* (Ubuntu 24.04)

**Symptôme :** malgré `PasswordAuthentication no`, le mot de passe restait **accepté**.

**Diagnostic** avec la config réellement appliquée :
```bash
sudo sshd -T | grep -iE "passwordauthentication|permitrootlogin|pubkeyauthentication"
# → passwordauthentication yes   ❌ (alors qu'on a écrit "no")
```
**Cause :** SSH lit les fichiers de `sshd_config.d/` dans l'ordre alphabétique et, pour chaque réglage, **le premier vu l'emporte**. Le fichier `50-cloud-init.conf` (déposé par l'installeur) imposait `PasswordAuthentication yes` et passait **avant** mon `99-durcissement.conf`.

**Correctif :**
```bash
sudo sed -i 's/^/#/' /etc/ssh/sshd_config.d/50-cloud-init.conf   # neutraliser l'intrus
sudo rm /etc/ssh/sshd_config.d/99-durcissement.conf.save         # nettoyer un résidu
sudo sshd -t && sudo systemctl reload ssh
```

---

## ✅ Tests & preuves

| Test | Commande | Résultat attendu | Statut |
|---|---|---|---|
| Connexion par clé | `ssh lisow@192.168.1.35` | Entrée **sans mot de passe** | ✅ |
| Config appliquée | `sudo sshd -T \| grep -i passwordauthentication` | `passwordauthentication no` | ✅ |
| Mot de passe refusé | `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no lisow@192.168.1.35` | `Permission denied (publickey)` | ✅ |
| Root désactivé | `sudo sshd -T \| grep -i permitrootlogin` | `permitrootlogin no` | ✅ |

---

## 🧠 Compétences démontrées (lien RNCP 37680)

- Mise en place d'une **authentification forte** (clé SSH `ed25519`) et suppression de l'auth par mot de passe.
- **Durcissement** d'un service exposé selon les bonnes pratiques (ANSSI).
- **Diagnostic** de configuration en production (`sshd -T`) et résolution d'un conflit de fichiers de configuration.
- Méthode d'admin sécurisée : validation de syntaxe avant application, session de secours pour éviter l'auto-verrouillage.

---

## 📂 Contenu du dépôt

| Fichier | Description |
|---|---|
| `Aide-memoire-AIS.md` | Mémo des commandes (Blocs 1 & 2, Git, **Bloc 3 SSH**) |
| `README.md` | Ce document |

---

> 🔜 **Pour aller plus loin (suite du Bloc 3) :** pare-feu **UFW** (*deny by default*, ouvrir seulement le port 22), **Fail2Ban** (bannissement auto du bruteforce SSH), sensibilisation **ANSSI/RGPD**.
