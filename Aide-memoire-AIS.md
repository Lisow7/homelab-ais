# 🛠️ Aide-mémoire AIS — Administration Linux

> Guide de référence personnel — **Lisow**
> Objectif : alternance *Administrateur d'Infrastructures Sécurisées* (AIS)
> Document vivant : on ajoute un bloc à chaque module terminé.

---

## 📑 Sommaire

- [Bloc 1 — Monter & administrer un serveur Ubuntu](#bloc-1) ✅ **TERMINÉ**
- [Bloc 2 — Réseaux](#bloc-2) ✅ **TERMINÉ**
- [Git & GitHub](#git) ✅ **TERMINÉ**
- [Bloc 3 — Sécurité (SSH par clé + durcissement)](#bloc-3) 🚧 **EN COURS**

> **🧭 Réflexes transverses** (valables tout le temps) :
> - 🔇 **Le silence = succès.** Une commande qui réussit n'affiche souvent rien.
> - 🙈 **Mot de passe invisible = normal.** Rien ne s'affiche quand on tape, on continue.
> - 🔠 **Orthographe stricte.** Une lettre de travers = commande inconnue.
> - 🔡 **Tout en minuscules** pour les noms d'utilisateurs et de groupes.
> - ⌨️ **`Tab`** = autocomplète (tape `ls -l sauv` + Tab → nom exact, zéro faute). **`↑`** = rappelle la dernière commande.
> - 🖱️ Dans VirtualBox : **`CTRL DROITE`** libère la souris/clavier de la VM.

---

<a name="bloc-1"></a>
## BLOC 1 — Monter & administrer un serveur Ubuntu ✅

### 🎯 Ma phrase d'entretien
> « J'administre un serveur Ubuntu Server (monté de A à Z dans VirtualBox) en ligne de commande : mises à jour système, utilisateurs, groupes, permissions, processus, services — et j'automatise des tâches avec des scripts Bash. »

---

### 1️⃣ Installer le serveur (les pièges à connaître)

| Étape | Le bon réflexe |
|---|---|
| Virtualisation | Activer **SVM Mode** dans le BIOS (AMD) → sinon pas de VM 64-bit |
| Création VM | **Décocher** « Unattended Installation » → on installe à la main |
| Type d'install | **Ubuntu Server** (pas *minimized*), rien dans *third-party drivers* |
| SSH | ✅ **Cocher** « Install OpenSSH server » (Espace) → pilotage à distance |
| Snaps | **Ne rien installer**, serveur propre et minimal |
| Au 1er reboot | Retirer l'ISO : **Périphériques → Lecteurs optiques → décocher** |

**Navigation clavier dans l'installeur (pas de souris) :**
`↑ ↓` bouger · `Tab` changer de zone · `Entrée` valider · `Espace` cocher · `[ Back ]` revenir

---

### 2️⃣ Entretien système — le trio de base

```bash
sudo apt update       # repérer les mises à jour disponibles
sudo apt upgrade      # les installer
sudo apt autoremove   # nettoyer les paquets devenus inutiles
```

> 🧠 `sudo` = exécuter en administrateur · `apt` = gestionnaire de paquets Ubuntu.
> Toujours `update` **avant** `upgrade`. Confirmer avec `o`/`y` + Entrée.

---

### 3️⃣ Utilisateurs & groupes

| Commande | Rôle |
|---|---|
| `sudo groupadd equipe-dev` | Créer un **groupe** |
| `sudo adduser paul` | Créer un **utilisateur** (+ dossier perso, interactif) |
| `sudo usermod -aG equipe-dev paul` | **Ajouter** paul au groupe |
| `id paul` | Voir l'identité et les groupes d'un utilisateur |
| `getent group equipe-dev` | Vérifier qu'un groupe existe |
| `whoami` | Savoir en tant que qui je suis connecté |

> ⚠️ **Toujours `-aG` ensemble.** `-G` seul **écrase** tous les groupes ; le `-a` (*append*) **ajoute** sans effacer. Oublier le `-a` = erreur classique qui casse des accès.
> 🧠 `adduser` (convivial, crée le dossier perso) ≠ `useradd` (brut, à éviter pour débuter).

---

### 4️⃣ Navigation & manipulation de fichiers

| Commande | Rôle |
|---|---|
| `pwd` | Où suis-je ? *(Print Working Directory)* |
| `ls` | Lister le contenu du dossier |
| `ls -l` | Liste **détaillée** (droits, proprio, groupe…) — `l` = lettre L ! |
| `cd projets` | Entrer dans un dossier |
| `cd ..` | Remonter d'un niveau |
| `cd` (seul) | Retour direct à mon dossier perso |
| `mkdir projets` | Créer un dossier |
| `touch fichier.txt` | Créer un fichier vide |
| `nano fichier.txt` | Éditer un fichier *(sauver : `Ctrl+O` `Entrée` · quitter : `Ctrl+X`)* |
| `cat fichier.txt` | Afficher le contenu d'un fichier |
| `cp source dest` | **Copier** un fichier *(ajouter `-r` pour un dossier)* |
| `mv source dest` | **Déplacer** OU **renommer** un fichier |
| `rm fichier` | **Supprimer** ⚠️ définitif, pas de corbeille ! |

> 🎨 Couleurs de `ls` : **bleu** = dossier · blanc = fichier · **vert** = exécutable.
> ⛔ **`rm` est dangereux** : suppression immédiate et irréversible. **Ne JAMAIS taper `rm -rf /`** (efface tout le système). Relire avant Entrée ; `rm -i` demande confirmation.

---

### 5️⃣ Permissions (le cœur de la sécurité)

**Lire une ligne `ls -l` :**
```
d rwx rwx ---   root  equipe-dev   partage-dev
│  │   │   │     │        │
│  │   │   │     │        └─ groupe propriétaire
│  │   │   │     └────────── utilisateur propriétaire
│  │   │   └─ AUTRES : aucun droit
│  │   └───── GROUPE : rwx (lire/écrire/entrer)
│  └───────── PROPRIÉTAIRE : rwx
└──────────── d = dossier (- = fichier)
```

**Changer le propriétaire / groupe :**
```bash
sudo chown root:equipe-dev /srv/partage-dev   # format proprio:groupe
```

**Changer les droits (méthode chiffrée) :**
```bash
sudo chmod 770 /srv/partage-dev   # 7=rwx 0=rien → proprio rwx / groupe rwx / autres rien
chmod +x sauvegarde.sh            # méthode rapide : ajoute juste le droit d'exécution
```
> 🔢 r=**4**, w=**2**, x=**1** → on additionne par catégorie. `7`=rwx · `0`=rien.

**Exercice type — créer un dossier partagé d'équipe :**
```bash
sudo mkdir /srv/partage-dev
sudo chown root:equipe-dev /srv/partage-dev   # donner au groupe
sudo chmod 770 /srv/partage-dev               # groupe peut écrire, autres rien
```

---

### 6️⃣ Processus (ce qui tourne sur la machine)

| Commande | Rôle |
|---|---|
| `ps` | Mes processus (session courante) |
| `ps aux` | **Tous** les processus de la machine (+ PID, CPU, RAM) |
| `top` | Tableau de bord temps réel *(quitter : `q`)* |
| `kill <PID>` | Arrêter proprement un processus par son numéro |
| `kill -9 <PID>` | Arrêt **brutal** (dernier recours seulement) |

> 🧠 Réflexe : programme bloqué → trouver son **PID** (`ps`/`top`) → `kill PID`.
> Test sans risque : `sleep 300 &` lance un cobaye en fond, puis on le `kill`.

---

### 7️⃣ Services (systemctl)

| Commande | Effet |
|---|---|
| `systemctl status ssh` | Voir l'état d'un service *(quitter : `q`)* |
| `sudo systemctl start ssh` | Démarrer maintenant |
| `sudo systemctl stop ssh` | Arrêter |
| `sudo systemctl enable ssh` | Démarrage **automatique** au boot |

> 🧠 **États à lire** : `active (running)` 🟢 = tourne · `inactive (dead)` = arrêté · `disabled` = ne démarre pas au boot.
> 🧠 Un service = un processus géré par `systemctl`. **SSH écoute sur le port 22.**
> 💡 Ubuntu peut mettre SSH en veille via `ssh.socket` (réveil à la 1ʳᵉ connexion) — c'est normal.

---

### 8️⃣ Scripting Bash (automatisation)

**Créer le script** (`nano sauvegarde.sh`) :
```bash
#!/bin/bash
echo "Debut de la sauvegarde..."
cp -r /home/lisow/projets /home/lisow/projets-backup
echo "Sauvegarde terminee !"
```

**Le rendre exécutable et le lancer :**
```bash
chmod +x sauvegarde.sh   # ajoute le droit d'exécution (sinon il ne tourne pas)
./sauvegarde.sh          # le ./ = "exécute le script du dossier courant"
```

> 🧠 `#!/bin/bash` = le **shebang**, toujours en 1ʳᵉ ligne (« exécute-moi avec bash »).
> `echo` = afficher un message. Un script = une suite de commandes lancée d'un coup.

---

### 9️⃣ Tester un accès en tant qu'un autre utilisateur

```bash
su - paul                          # devenir paul (mot de passe DE paul)
id                                 # vérifier ses groupes
touch /srv/partage-dev/test.txt    # tester l'écriture (silence = OK)
exit                               # revenir à mon compte
```

> 🧠 `su` = *switch user*. Le test prouve que groupe + chown + chmod fonctionnent vraiment.

---

### 🐞 Mémo des erreurs vécues (et leur cause)

| Message / symptôme | Cause | Correction |
|---|---|---|
| `Command 'sudp' not found` | Faute de frappe | Réécrire `sudo` |
| `username matching regular expression` | Majuscule (`Paul`) | Tout en minuscules : `paul` |
| `ls -1` n'affiche pas les détails | Chiffre `1` au lieu de la lettre `l` | `ls -l` (lettre L) |
| `cannot access 'sauvegardde.sh'` | Faute de frappe dans un nom | Utiliser **`Tab`** pour autocompléter |
| Prompt affiche `/home$` | `cd ..` de trop | `cd` pour rentrer chez soi |
| `Please remove the installation medium` | ISO encore montée au reboot | Périphériques → Lecteurs optiques → décocher |
| `watchdog: soft lockup` | VM qui rame un instant | Sans gravité, ignorer |

---

### 📌 Infrastructure de référence (ma VM)

| Élément | Valeur |
|---|---|
| OS | Ubuntu Server 24.04 LTS (*Noble*) |
| Hôte | VirtualBox 7.2.x — Windows / AMD Ryzen 5 5600X, 16 Go |
| Ressources VM | 4096 Mo RAM · 2 CPU · disque 25 Go (LVM) |
| Hostname | `serveur-ais` |
| Utilisateur | `lisow` (+ `paul` créé pour tests) · Groupe : `equipe-dev` |
| SSH | OpenSSH installé, service **enabled** ✅ (port 22) |
| Arrêt propre | `sudo poweroff` |

> 🔜 **Points de reprise / prochaines étapes :**
> - Passer le réseau VM en **Pont** (Paramètres → Réseau) puis se **connecter en SSH depuis Windows**
> - Démarrer **Git & GitHub** (versionner et publier le homelab)

---
---

<a name="git"></a>
## GIT & GITHUB ✅

### 🎯 Ma phrase d'entretien
> « Je versionne mes projets avec Git et je les publie sur GitHub : j'installe et configure Git, je crée un dépôt, et j'utilise le cycle **add → commit → push** pour suivre et partager mon travail. Mon portfolio est public sur GitHub. »

---

### 🧠 L'idée en une image

- **Git** = un « historique de sauvegardes » de mon projet, sur **mon PC**.
- **GitHub** = le coffre **en ligne** où je publie ce projet (visible des recruteurs si **public**).

| Mot | Sens |
|---|---|
| **dépôt** (*repo*) | Le dossier de projet versionné |
| **commit** | Une sauvegarde datée et signée, avec un message |
| **branche `main`** | La ligne de travail principale (par défaut) |
| **`(main)` dans le prompt** | Repère Git Bash : « je suis **dans** un dépôt Git » |

---

### 1️⃣ Installation & configuration (une seule fois)

| Commande | Rôle |
|---|---|
| `git --version` | Vérifier que Git est installé |
| `git config --global user.name "Prénom Nom"` | Mon identité (signe mes commits) |
| `git config --global user.email "mail@github"` | ⚠️ **l'email du compte GitHub** (rattache les commits au profil) |
| `git config --global --list` | Vérifier ma config |

> 🧠 Sous Windows, on travaille dans **Git Bash** = un terminal façon Linux (`cd`, `ls`, `pwd` y marchent).

---

### 2️⃣ Le cycle Git fondamental (à graver — 90% de l'usage)

> 📥 `clone` → ✏️ *je modifie* → ➕ `add` → 💾 `commit` → 📤 `push`

| Commande | Rôle |
|---|---|
| `git clone <url>` | Rapatrier un repo GitHub sur mon PC (1ʳᵉ fois : connexion par navigateur) |
| `git status` | Où j'en suis ? (**rouge** = non suivi · **vert** = prêt à committer) |
| `git add .` | Sélectionner **tous** les changements (le `.` = tout) |
| `git commit -m "message clair"` | Enregistrer une sauvegarde avec un message |
| `git push` | Envoyer mes commits vers GitHub |
| `git pull` | Récupérer les changements **depuis** GitHub |
| `git log --oneline` | Voir l'historique des commits |
| `git mv ancien nouveau` | **Renommer** un fichier proprement (Git comprend que c'est un renommage) |

---

### 3️⃣ Créer un dépôt sur GitHub

`+` (haut-droite) → **New repository** → nom en **minuscules-avec-tirets** → **Public** ✅ → cocher **Add a README** → **Create repository**.

> 🧠 Public = visible des recruteurs (c'est le but du portfolio). Privé = personne ne le voit.

---

### 🐞 Mémo des erreurs / pièges vécus (Git)

| Message / symptôme | Cause | Correction |
|---|---|---|
| `warning: LF will be replaced by CRLF` | Windows et Linux gèrent les **fins de ligne** différemment | **Normal**, Git harmonise sans rien casser → ignorer |
| Noms de fichiers avec **espaces** | `Bloc 2.md` → oblige des guillemets `"…"` et complique les liens | Renommer en **`tirets`** : `git mv "Bloc 2.md" "Bloc-2.md"` |
| GitHub **refuse le mot de passe** au `push` | L'auth par mot de passe n'existe plus | Connexion par **navigateur** (Credential Manager) ou *token* |
| Le prompt n'affiche pas `(main)` | Je ne suis pas dans le dossier du repo | `cd` dans le dossier cloné |

---

### 📌 Infrastructure de référence & points de reprise (Git)

| Élément | Valeur |
|---|---|
| Dépôt | `github.com/Lisow7/homelab-ais` (**public**) |
| Contenu | `Bloc-1_Homelab-Linux.md` · `Bloc-2_Labo-Reseaux-Packet-Tracer.md` · `Aide-memoire-AIS.md` · `Labo-Reseaux-AIS.pkt` |
| Copie locale | `C:\Users\Antoi\Desktop\homelab-ais` |

> 🔜 **Points de reprise / prochaines étapes :**
> - Ajouter un dossier `captures/` avec les screenshots du labo (topologie + ping).
> - Réutiliser le cycle `add → commit → push` à chaque nouveau livrable de bloc.

---
---

<a name="bloc-2"></a>
## BLOC 2 — RÉSEAUX ✅

### 🎯 Ma phrase d'entretien
> « Je comprends comment les machines communiquent : adressage IP et masques, découpage en sous-réseaux, résolution DNS, attribution automatique par DHCP, et routage entre réseaux. Je sais le diagnostiquer en ligne de commande (`ip a`, `dig`, `ip route`, `traceroute`) et le maquetter dans Packet Tracer. »

---

### 1️⃣ Le modèle TCP/IP (la pile en 4 couches)

Le réseau, c'est la Poste : pour qu'une lettre arrive, 4 niveaux coopèrent.

| Couche | Rôle | Analogie postale |
|---|---|---|
| **Application** | Le contenu (HTTP, DNS…) | Le texte de la lettre |
| **Transport** | **TCP** (fiable, accusé de réception) ou **UDP** (rapide, sans garantie) | Recommandé vs courrier simple |
| **Internet** | L'adressage et le routage (**IP**) | L'adresse sur l'enveloppe |
| **Accès réseau** | Le support physique (Ethernet, Wi-Fi, **MAC**) | Le facteur, le camion |

> 🧠 Chaque machine a une **adresse IP**. **TCP** garantit que tout arrive ; **UDP** va vite sans garantie.

---

### 2️⃣ Adresse IP & masque

- Une **IP** = 4 octets = **32 bits**. Le **masque** sépare la *partie réseau* (le quartier) de la *partie machine* (la maison).
- **`/24`** = `255.255.255.0` = **3 octets réseau + 1 octet machine**. (`/16` = 2+2 · `/8` = 1+3.)
- Là où le masque a un **`1`** (binaire) → bit figé = réseau · un **`0`** → bit libre = machine.

**Déduire un réseau à partir d'un `/24` (ex. `10.0.2.15/24`) :**

| Question | Méthode | Résultat |
|---|---|---|
| Adresse réseau | partie machine = **0** | `10.0.2.0` |
| Broadcast | partie machine = **max (255)** | `10.0.2.255` |
| 1ʳᵉ machine | réseau **+1** | `10.0.2.1` |
| Dernière machine | broadcast **−1** | `10.0.2.254` |
| Nb de machines | **2ⁿ − 2** (on retire réseau + broadcast) | `254` |

| Commande | Rôle |
|---|---|
| `ip a` | Voir mes adresses IP (`inet` = IPv4 · `brd` = broadcast · `dynamic` = donnée par DHCP) |
| `ping -c 4 8.8.8.8` | Tester la connexion (`-c 4` = 4 paquets, sinon infini → `Ctrl+C`) |
| `ipcalc 10.0.2.15/24` | Calculer réseau / broadcast / plage / nb de machines |

---

### 3️⃣ Sous-réseaux (subnetting)

Découper un réseau = **emprunter des bits** à la partie machine pour créer des sous-réseaux.

> 🔢 **Astuce du pas (taille de bloc) = `256 − dernier octet du masque`.**
> Ex. `/26` → masque `…192` → pas = `256 − 192 = 64` → sous-réseaux qui démarrent en `.0`, `.64`, `.128`, `.192`.

| Notation | Masque | Pas | Sous-réseaux dans un /24 | Machines/bloc |
|---|---|---|---|---|
| `/25` | `…128` | 128 | 2 | 126 |
| `/26` | `…192` | 64 | 4 | 62 |
| `/27` | `…224` | 32 | 8 | 30 |
| `/28` | `…240` | 16 | 16 | 14 |

> 🧠 Dans chaque bloc : réseau + broadcast = 2 adresses **non utilisables** → toujours `−2`.

---

### 4️⃣ DNS — l'annuaire (nom → IP)

- Traduit `google.com` → `172.217.22.46`. **Hiérarchique** : résolveur → racine `.` → TLD `.com` → serveur **autoritaire**.
- **Cache + TTL** : la réponse est gardée un temps (le TTL en secondes). 2ᵉ requête identique = quasi instantanée (`Query time: 0 msec`), et le TTL **diminue** entre deux appels.
- **Le DNS écoute sur le port 53.**

| Commande | Rôle |
|---|---|
| `cat /etc/resolv.conf` | Voir mon serveur DNS *(souvent `127.0.0.53` = relais local systemd-resolved, normal)* |
| `sudo apt install -y dnsutils` | Installer l'outil `dig` |
| `dig google.com` | Interroger le DNS · lire `ANSWER SECTION` (`A` = IPv4), le `TTL` et `Query time` |

---

### 5️⃣ DHCP — l'attribution automatique d'adresses

- Distribue les IP **automatiquement** (sinon = config manuelle de chaque poste). Le mot **`dynamic`** dans `ip a` = adresse fournie par DHCP.
- Échange en 4 temps : **DORA** = **D**iscover (broadcast « il me faut une IP ») → **O**ffer → **R**equest → **A**ck.
- Une IP DHCP = une **location** (un *bail* / lease à durée limitée, à renouveler). Le DHCP livre le **kit complet** : IP + masque + **passerelle** + DNS.

| Commande | Rôle |
|---|---|
| `networkctl status enp0s3` | Voir l'adresse `(DHCP4 via …)`, la `Gateway`, le `DNS` — tout ce que le DHCP a fourni |

---

### 6️⃣ Routage — comment un paquet trouve son chemin

**La décision, prise pour chaque paquet :** destination dans **mon** réseau → **envoi direct** · sinon → **envoi à la passerelle**, qui relaie de routeur en routeur.

| Commande | Rôle |
|---|---|
| `ip route` | Lire ma table de routage |

> 🧠 Lecture de `ip route` :
> - `default via 10.0.2.2 …` → **route par défaut** : « pour tout le reste, passe par la passerelle » (branche *ailleurs*).
> - `10.0.2.0/24 dev enp0s3 scope link` → « mon propre quartier, j'envoie en direct » (branche *même réseau*).
> - `proto dhcp` = route installée par le DHCP · `proto kernel` = route déduite par le système.

---

### 7️⃣ traceroute — voir les sauts

- Révèle le chemin **saut par saut** (*hop*). Astuce : envoie des paquets au **TTL croissant** (1, 2, 3…) ; chaque routeur où un paquet expire se signale.
- Le **TTL** diminue de **1 à chaque routeur traversé** (rappel : `ttl=64` au départ d'un ping).

| Commande | Rôle |
|---|---|
| `sudo apt install -y traceroute` | Installer l'outil |
| `traceroute -n 8.8.8.8` | Tracer le chemin (`-n` = afficher les IP sans résolution de noms) |

> ⚠️ **En NAT VirtualBox**, `traceroute` n'affiche que le 1ᵉʳ saut (la passerelle) puis des `* * *` : c'est une **limite du mode NAT**, pas une erreur. En **Pont**, on verrait les vrais routeurs.
> 🧠 **Réflexe d'admin :** un outil réseau qui se comporte bizarrement dans une VM → soupçonner d'abord le **mode réseau de la VM**.

---

### 🐞 Mémo des erreurs vécues (Bloc 2)

| Message / symptôme | Cause | Correction |
|---|---|---|
| `ig: command not found` | Faute de frappe (`d` manquant) | `dig` |
| `ping: .8.8.8.8: Name or service not known` | Un **point en trop** devant l'IP → pris pour un nom de machine | Retirer le point : `8.8.8.8` |
| `Interface "en0s3" not found` | `p` manquant dans le nom de carte | `enp0s3` (+ `Tab` pour autocompléter) |
| `traceroute` → `* * *` partout sauf saut 1 | Limite du mode **NAT** de VirtualBox | Normal ; passer en **Pont** pour voir le vrai chemin |
| `watchdog: BUG: soft lockup` | VM qui rame un instant | Sans gravité, ignorer |

---

### 📌 Infrastructure de référence & points de reprise (Bloc 2)

| Élément | Valeur |
|---|---|
| Labo Packet Tracer | 2 réseaux reliés par un routeur **1941** |
| Réseau A | `192.168.1.0/24` — passerelle `192.168.1.1` (routeur G0/0) — PC0 `.10`, PC1 `.11` |
| Réseau B | `192.168.2.0/24` — passerelle `192.168.2.1` (routeur G0/1) — PC2 `.10` |
| Preuve de routage | `ping` inter-réseau → `TTL=127` (−1 = **1 routeur traversé**) |
| Fichier | `Labo-Reseaux-AIS.pkt` |

> 🔜 **Points de reprise / prochaines étapes :**
> - **Git & GitHub** : publier le homelab Linux + ce labo réseau (en cours).
> - Passer la VM en **Pont** puis **SSH depuis Windows** (début du Bloc 3) ; en profiter pour relancer `traceroute` et voir les vrais sauts.
> - Bloc 3 — Sécurité Linux (SSH par clé, UFW, Fail2Ban, ANSSI/RGPD).

---
---

<a name="bloc-3"></a>
## BLOC 3 — SÉCURITÉ 🚧

> Sous-parties : **SSH par clé** ✅ · **Durcissement SSH** ✅ · UFW (à venir) · Fail2Ban (à venir) · ANSSI/RGPD (à venir).

### 🎯 Ma phrase d'entretien
> « Je sécurise un serveur Linux exposé : accès SSH par clé uniquement (root désactivé, mot de passe refusé), dans une démarche alignée sur les bonnes pratiques de l'ANSSI. Je sais diagnostiquer la config réellement appliquée avec `sshd -T` et résoudre les conflits de fichiers *drop-in*. »

---

### 1️⃣ Passer la VM en réseau Pont (prérequis)

Le mode **NAT** cache la VM derrière un réseau privé (`10.0.2.x`) → injoignable depuis Windows. Le mode **Pont** la branche sur le vrai réseau local → elle reçoit une IP de la box (ex. `192.168.1.35`), comme un poste physique.

| Étape | Action |
|---|---|
| Éteindre la VM | `sudo poweroff` |
| VirtualBox | `serveur-ais` → Configuration → **Réseau** → Carte 1 |
| Mode | **Accès par pont** *(Bridged Adapter)* |
| Nom | la **carte physique active** (Ethernet `Realtek...` ou Wi-Fi) |
| Vérifier (dans la VM) | `ip a` → chercher `inet 192.168.x.x` sur `enp0s3` |

> 🧠 Bonus : le mode Pont **répare aussi `traceroute`** (qui ne voyait qu'un saut en NAT).

---

### 2️⃣ Authentification par clé SSH (depuis Windows)

**Le principe :** une paire de clés liées. La **privée** reste secrète sur Windows (ne sort jamais). La **publique** (copiable) est déposée sur la VM. Au login, la VM lance un défi que **seule la clé privée peut résoudre** — aucun secret ne circule, plus de mot de passe.

| Commande (PowerShell, côté Windows) | Rôle |
|---|---|
| `ssh-keygen -t ed25519` | Générer la paire (`ed25519` = algo moderne recommandé) |
| `cat $env:USERPROFILE\.ssh\id_ed25519.pub` | Afficher la clé **publique** (faite pour être partagée) |
| `ssh lisow@<IP>` | Se connecter (1ʳᵉ fois : taper `yes` pour enregistrer l'empreinte) |

**Déposer la clé publique sur la VM (en une commande) :**
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh lisow@192.168.1.35 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```
> 🧠 `>>` **ajoute** au fichier `authorized_keys` (sans écraser). Les `chmod 700/600` sont **obligatoires** : SSH **refuse** les clés si les droits du dossier `.ssh` sont trop ouverts.
> 🧠 `~/.ssh/known_hosts` (côté Windows) : empreinte de chaque serveur connu. Si elle **change** sans raison → SSH bloque (protection anti-usurpation / MITM).

---

### 3️⃣ Durcissement SSH (fermer les portes)

But : **condamner le mot de passe et le compte root**, ne laisser que la clé.

**Écrire la config sans éditeur** (drop-in dédié, on ne touche pas au fichier d'origine) :
```bash
sudo tee /etc/ssh/sshd_config.d/99-durcissement.conf > /dev/null <<'EOF'
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
EOF
```

| Directive | Effet |
|---|---|
| `PermitRootLogin no` | Interdit la connexion directe en **root** |
| `PasswordAuthentication no` | Refuse l'auth par **mot de passe** |
| `PubkeyAuthentication yes` | Autorise l'auth par **clé** |
| `KbdInteractiveAuthentication no` | Coupe le canal **clavier-interactif** (sinon le mot de passe peut passer par là) |

**Toujours dans cet ordre — valider AVANT d'appliquer :**
```bash
sudo sshd -t                    # test syntaxe (silence = OK)
sudo systemctl reload ssh       # appliquer (reload ne coupe PAS les sessions en cours)
```

> ⚠️ **Règle d'or :** ne JAMAIS fermer la session SSH actuelle avant d'avoir vérifié dans une **2ᵉ fenêtre** que la nouvelle config marche. C'est le filet de sécurité contre l'auto-verrouillage.

**Vérifier la config RÉELLEMENT appliquée** (commande reine) :
```bash
sudo sshd -T | grep -iE "passwordauthentication|permitrootlogin|kbdinteractive|pubkeyauthentication"
```

**Preuves de bon fonctionnement :**
```bash
ssh lisow@192.168.1.35                                  # → entre SANS mot de passe (clé)
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no lisow@192.168.1.35
# → doit répondre : Permission denied (publickey)  ✅ (mot de passe bien refusé)
```

---

### 🐞 Mémo des erreurs vécues (Bloc 3)

| Message / symptôme | Cause | Correction |
|---|---|---|
| nano fait « bip » et ne réagit pas | La fenêtre n'a pas le focus / touches interceptées | Fermer la fenêtre (rien n'est enregistré) → préférer `tee` au lieu de nano |
| `id_ed25519 already exists ... Overwrite (y/n)?` | Une clé existe déjà | Répondre **`n`** (ne PAS écraser, ça détruirait l'ancienne) — on la réutilise |
| `key already exists` au `ssh-keygen` | idem | Réutiliser la clé en place ; copier juste sa partie `.pub` |
| **Mot de passe ACCEPTÉ malgré `PasswordAuthentication no`** | Conflit de drop-ins : `50-cloud-init.conf` (`PasswordAuthentication yes`) gagne car **premier vu = gagnant** (ordre alphabétique) | Neutraliser l'intrus : `sudo sed -i 's/^/#/' /etc/ssh/sshd_config.d/50-cloud-init.conf` puis reload |
| `99-durcissement.conf.save` traîne | Résidu laissé par nano | `sudo rm` du fichier `.save` (SSH risque de le lire) |
| `sudo: Sorry, try again` | Faute de frappe dans le mot de passe `sudo` | Retaper (invisible) ; `Ctrl+C` pour annuler |

> 🧠 **LA leçon du bloc :** `sudo sshd -T` montre la config *effectivement* appliquée après lecture de **tous** les fichiers `sshd_config.d/*.conf`. Pour un réglage, **le premier fichier vu (ordre alpha) l'emporte** — d'où l'intérêt du préfixe numérique (`50-`, `99-`).

---

### 📌 Infrastructure de référence & points de reprise (Bloc 3)

| Élément | Valeur |
|---|---|
| Réseau VM | **Pont** sur Ethernet `Realtek` — IP `192.168.1.35/24` (DHCP box) |
| Accès | SSH **par clé `ed25519`** depuis Windows (PowerShell / OpenSSH natif) |
| Config durcie | `/etc/ssh/sshd_config.d/99-durcissement.conf` (+ `50-cloud-init.conf` neutralisé) |
| État vérifié | root `no` · mot de passe `no` · clé `yes` · `Permission denied (publickey)` ✅ |

> 🔜 **Points de reprise / prochaines étapes :**
> - **UFW** : pare-feu en politique *deny by default*, n'ouvrir que le port **22** (SSH).
> - **Fail2Ban** : bannir les IP qui forcent SSH *(note Ubuntu 24.04 : la jail `sshd` est déjà en backend `systemd`/journald par défaut)*.
> - **ANSSI / RGPD** : culture sécu (recommandations d'hygiène, durcissement, journalisation).

---
---

> *Structure réutilisable pour chaque nouveau bloc : 🎯 phrase d'entretien · tableaux de commandes · 🐞 mémo des erreurs · 📌 points de reprise.*
