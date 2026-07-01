# 🔒 Sécurité — Serveur Linux durci (SSH par clé, UFW, Fail2Ban)

> Projet réalisé dans le cadre de ma préparation au **Titre Professionnel Administrateur d'Infrastructures Sécurisées (AIS)** — RNCP 37680.
> Outil : **Ubuntu Server 24.04 LTS** (VM VirtualBox, réseau Pont) · Client : **OpenSSH (Windows)** · Auteur : **Lisow**

---

## 🎯 Objectif

Sécuriser l'accès distant à un serveur Linux et réduire sa surface d'attaque, en suivant les bonnes pratiques du **guide d'hygiène de l'ANSSI** : authentification SSH par clé uniquement, durcissement du service, pare-feu en politique *deny by default*, et protection anti-bruteforce.

> 💬 **Ma phrase d'entretien :** « J'ai sécurisé l'accès distant d'un serveur Linux en suivant le guide d'hygiène de l'ANSSI : authentification SSH par clé (recommandation OpenSSH de l'ANSSI), pare-feu UFW en *deny by default* pour réduire la surface d'attaque, et Fail2Ban pour la supervision et la réaction — le tout en défense en profondeur. Ces mesures répondent aussi à l'obligation de sécurité de l'article 32 du RGPD. »

---

## 🧱 Contexte

| Élément | Valeur |
|---|---|
| Serveur | Ubuntu Server 24.04 LTS — VM VirtualBox `serveur-ais` |
| Réseau | Mode **Pont** (Bridged) → IP `192.168.1.35/24` (DHCP box) |
| Poste d'administration | Windows — client **OpenSSH** natif (PowerShell) |
| Type de clé | **ed25519** (algorithme moderne recommandé) |

---

## 🔧 Réalisation

### 1. Réseau Pont (prérequis)
Passage de la carte réseau de **NAT** (réseau privé `10.0.2.x`, injoignable) à **Pont** : la VM reçoit une IP du réseau local et devient joignable depuis Windows.
Validation : `ip a` → `inet 192.168.1.35` ; `ping` depuis Windows → 0 % de perte, `TTL=64` (même réseau, aucun routeur traversé).

### 2. Authentification par clé SSH
```powershell
ssh-keygen -t ed25519    # génère la paire (clé privée jamais partagée)
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh lisow@192.168.1.35 `
  "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```
Validation : `ssh lisow@192.168.1.35` se connecte **sans mot de passe**.

### 3. Durcissement SSH
Drop-in dédié `/etc/ssh/sshd_config.d/99-durcissement.conf` :
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
```
```bash
sudo sshd -t && sudo systemctl reload ssh   # tester la syntaxe AVANT d'appliquer
```
Bonne pratique : garder une 2ᵉ session SSH ouverte (filet) pendant le rechargement.

### 4. Pare-feu UFW (deny by default)
Ordre critique : **autoriser SSH avant d'activer** (sinon auto-verrouillage).
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status verbose   # Status: active — seul 22/tcp (OpenSSH) ALLOW IN
```

### 5. Fail2Ban (anti-bruteforce)
Pré-configuré sur Ubuntu 24.04 (jail `sshd` active, backend `journald`). Personnalisation via `jail.local` :
```
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24   # ne jamais se bannir soi-meme
maxretry = 4
findtime = 10m
bantime  = 1h
bantime.increment = true
[sshd]
enabled = true
```
```bash
sudo fail2ban-client -t && sudo systemctl reload fail2ban
sudo fail2ban-client status sshd            # jail active, Journal matches présent
```

---

## 🐞 Incident rencontré & résolu — conflit de *drop-in* (Ubuntu 24.04)

**Symptôme :** malgré `PasswordAuthentication no`, le mot de passe restait **accepté**.

**Diagnostic** (config réellement appliquée) :
```bash
sudo sshd -T | grep -i passwordauthentication   # → yes (au lieu de no)
```
**Cause :** SSH lit les fichiers de `sshd_config.d/` par ordre alphabétique et, pour chaque réglage, **le premier vu l'emporte**. Le fichier `50-cloud-init.conf` (installé par défaut) forçait `PasswordAuthentication yes` et passait **avant** mon `99-durcissement.conf`.

**Correctif :**
```bash
sudo sed -i 's/^/#/' /etc/ssh/sshd_config.d/50-cloud-init.conf
sudo rm /etc/ssh/sshd_config.d/99-durcissement.conf.save
sudo sshd -t && sudo systemctl reload ssh
```

---

## ✅ Tests & preuves

| Test | Commande | Résultat | Statut |
|---|---|---|---|
| Connexion par clé | `ssh lisow@192.168.1.35` | Entrée **sans mot de passe** | ✅ |
| Mot de passe refusé | `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no lisow@192.168.1.35` | `Permission denied (publickey)` | ✅ |
| Root désactivé | `sudo sshd -T \| grep -i permitrootlogin` | `permitrootlogin no` | ✅ |
| Pare-feu actif | `sudo ufw status verbose` | `Status: active` — seul `22/tcp` ouvert | ✅ |
| Fail2Ban actif | `sudo fail2ban-client status sshd` | jail `sshd`, `journald`, ban/unban testés | ✅ |

---

## 🛡️ Alignement ANSSI / RGPD

| Action réalisée | Principe du guide d'hygiène ANSSI |
|---|---|
| SSH par clé, root off | Authentifier & contrôler les accès (authentification forte) |
| UFW deny by default | Sécuriser le réseau (réduire la surface d'attaque) |
| Fail2Ban + journald | Superviser, auditer, réagir (journalisation) |
| L'ensemble empilé | Défense en profondeur |

Référence : note technique ANSSI « Recommandations pour un usage sécurisé d'(Open)SSH » (2016).
RGPD : ces mesures techniques répondent à l'obligation de sécurité du traitement (art. 32) ; notification CNIL sous 72 h en cas de violation (art. 33).

---

## 🧠 Compétences démontrées (lien RNCP 37680)

- Authentification forte (clé SSH `ed25519`) et suppression de l'auth par mot de passe.
- Durcissement d'un service exposé et pare-feu en politique *deny by default*.
- Protection anti-bruteforce et supervision (Fail2Ban / journald).
- Diagnostic de configuration en production (`sshd -T`) et résolution d'un conflit de drop-ins.
- Culture réglementaire : guide d'hygiène ANSSI et obligations RGPD.

---

## 📂 Contenu du dépôt

| Fichier | Description |
|---|---|
| `Aide-memoire-AIS.md` | Mémo des commandes (Blocs 1, 2, Git, **Bloc 3 complet**) |
| `README.md` | Ce document |

---

> **Bloc 3 — Sécurité : terminé.** Prochaine étape : suite du programme des modules AIS.
