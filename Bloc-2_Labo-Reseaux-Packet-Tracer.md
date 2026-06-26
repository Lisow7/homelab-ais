# 🌐 Labo Réseaux — Routage inter-réseaux (Cisco Packet Tracer)

> Projet réalisé dans le cadre de ma préparation au **Titre Professionnel Administrateur d'Infrastructures Sécurisées (AIS)** — RNCP 37680.
> Outil : **Cisco Packet Tracer 9.0** · Auteur : **Lisow**

---

## 🎯 Objectif

Concevoir, configurer et **valider** une infrastructure composée de **deux réseaux locaux distincts reliés par un routeur**, afin de démontrer la maîtrise de :
- l'**adressage IPv4** et des **masques de sous-réseau** ;
- la **séparation en sous-réseaux** (deux quartiers logiques distincts) ;
- le **routage inter-réseaux** (la communication entre deux réseaux via une passerelle).

---

## 🧱 Topologie

```
   RÉSEAU A — 192.168.1.0/24            RÉSEAU B — 192.168.2.0/24
   ┌───────────────────────┐           ┌───────────────────────┐
   │  PC0  192.168.1.10     │           │                       │
   │      \                 │           │                       │
   │       [Switch0]──┐     │           │   ┌──[Switch1]── PC2   │
   │      /           │     │           │   │        192.168.2.10│
   │  PC1  192.168.1.11│    │           │   │                   │
   └──────────────────┼─────┘           └───┼───────────────────┘
                      │                     │
                G0/0  │  ┌───────────┐  │  G0/1
              .1.1 ───┴──│  Routeur  │──┴─── .2.1
                         │   (1941)  │
                         └───────────┘
```

*(Capture de la topologie Packet Tracer à insérer ici → `captures/topologie.png`.)*

---

## 📋 Plan d'adressage

| Équipement | Interface | Adresse IP | Masque | Passerelle |
|---|---|---|---|---|
| **PC0** | FastEthernet0 | `192.168.1.10` | `255.255.255.0` | `192.168.1.1` |
| **PC1** | FastEthernet0 | `192.168.1.11` | `255.255.255.0` | `192.168.1.1` |
| **PC2** | FastEthernet0 | `192.168.2.10` | `255.255.255.0` | `192.168.2.1` |
| **Routeur0** | GigabitEthernet0/0 | `192.168.1.1` | `255.255.255.0` | — |
| **Routeur0** | GigabitEthernet0/1 | `192.168.2.1` | `255.255.255.0` | — |

> 🧠 **Règle clé :** chaque PC a pour passerelle l'adresse du routeur **dans son propre réseau**. C'est elle qui permet de sortir du quartier.

---

## 🔧 Étapes de réalisation

1. **Placement** des équipements : 3 PC, 2 switches `2960`, 1 routeur `1941`.
2. **Câblage** en cuivre droit (*Copper Straight-Through*) : PC ↔ switch et switch ↔ routeur.
3. **Configuration IP statique** des 3 PC (`Desktop → IP Configuration`), passerelle comprise.
4. **Configuration du routeur** (`Config`) : adresse sur chaque interface `G0/0` et `G0/1`, puis **`Port Status → On`** pour les activer (équivalent du `no shutdown` Cisco).
5. **Tests de connectivité** (`ping`).

---

## ✅ Tests & preuves

| Test | Commande | Résultat | TTL | Interprétation |
|---|---|---|---|---|
| Intra-réseau (A → A) | `ping 192.168.1.11` depuis PC0 | `Reply`, 0% loss | **128** | Envoi **direct**, aucun routeur traversé |
| **Inter-réseau (A → B)** | `ping 192.168.2.10` depuis PC0 | `Reply` | **127** | Le paquet a **traversé le routeur** (TTL −1) |

> 🔎 **La preuve du routage** : le passage de `TTL=128` à `TTL=127` démontre mathématiquement que le paquet a franchi **un** routeur pour atteindre l'autre réseau. Un premier `Request timed out` au démarrage est normal (découverte ARP).

*(Capture du `Command Prompt` à insérer ici → `captures/ping-inter-reseaux.png`.)*

---

## 🧠 Compétences démontrées (lien RNCP 37680 — BC01)

- Conception d'un plan d'adressage IPv4 cohérent sur deux sous-réseaux.
- Configuration d'équipements (postes, commutateurs, routeur).
- Mise en œuvre et **validation** du routage inter-réseaux.
- Diagnostic réseau par la pratique (`ping`, lecture du TTL).

---

## 📂 Contenu du dépôt

| Fichier | Description |
|---|---|
| `Labo-Reseaux-AIS.pkt` | Le fichier Packet Tracer (ouvrable / rejouable) |
| `captures/` | Captures d'écran (topologie, configuration, tests) |
| `README.md` | Ce document |

---

> 🔜 **Pour aller plus loin :** ajouter un serveur DHCP sur le routeur, un serveur DNS, ou un 3ᵉ réseau — et documenter le routage statique correspondant.
