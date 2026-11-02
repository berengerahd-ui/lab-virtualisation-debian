# lab-virtualisation-debian




Taches: 
Vérifier VT-x/AMD-V activé dans le BIOS (OK) ; 
Installer VirtualBox (Ok) ; 
installer Debian en mode serveur, sans bureau, avec SSH server + utilitaires usuels (Ok) ; 
Ajouter 2 cartes réseau : NAT + réseau privé hôte (OK) ; 
snapshot « base-propre » (OK) ; 
premier login, hostnamectl, ip a (OK) 
notes d'installation (...).

---------------------------------------------------------------------------


#Autoriser les pings entrants (ICMP) sur windows

netsh advfirewall firewall add rule name="Autoriser ping ICMPv4" 
protocol=icmpv4:8,any dir=in action=allow


#Afficher les configurations réseau sur Debian
ip a


#Activer une interface Rseau sur debian
ip link set enp0s8 up
dhcpcd enp0s8


-----------------------------------------------------------



# Note d'installation : VM Debian serveur (VirtualBox)

**Date :** 02/11/2026
**Auteur :** [AHOUANYIDE Berenger]
**Objectif :** Mettre en place un serveur Debian minimal (sans bureau) dans une machine virtuelle, accessible depuis le PC hôte Windows.

## 1. Prérequis : virtualisation matérielle

- Vérification que **VT-x / AMD-V** est activé dans le BIOS/UEFI de la machine hôte.
- Contrôle possible depuis Windows : Gestionnaire des tâches > Performances > CPU > « Virtualisation : activée ».
- Résultat : [activée d'origine / activée manuellement dans le BIOS].

## 2. Installation de VirtualBox

- Version installée : Oracle VirtualBox [7.2.22 r175245].
- Hôte : Windows [11].
- Carte réseau hôte créée automatiquement : « Ethernet 3 » en **192.168.56.1/24** (réseau privé hôte).

## 3. Création de la VM et installation de Debian

| Paramètre | Valeur |
|---|---|
| Nom de la VM | srv-deb |
| Système | Debian [version] 64 bits |
| RAM | [2048] Mo |
| CPU | [1] |
| Disque | [20] Go (dynamique) |
| Nom d'hôte | svr-deb |
| Utilisateur | benk |

Choix à l'installation :
- Mode **serveur, sans environnement de bureau**.
- Logiciels sélectionnés : **serveur SSH** + **utilitaires usuels du système**.
- Partitionnement : [guidé, disque entier / autre].
- Mot de passe root défini pendant l'installation (`sudo` non installé par défaut, on passe par `su -`).

## 4. Configuration réseau (2 cartes)

| Carte | Mode VirtualBox | Interface Debian | Adresse |
|---|---|---|---|
| Carte 1 | NAT | enp0s3 | 10.0.2.15/24 (DHCP) |
| Carte 2 | Réseau privé hôte | enp0s8 | 192.168.56.101/24 (DHCP) |

- **enp0s3 (NAT)** : donne l'accès à Internet à la VM (mises à jour, paquets). La VM n'est pas joignable directement depuis l'extérieur.
- **enp0s8 (réseau privé hôte)** : permet la communication directe PC Windows ↔ VM (SSH, tests).

Problème rencontré : enp0s8 était à l'état DOWN après l'installation. Résolution, en root :

```
ip link set enp0s8 up
dhcpcd enp0s8
```

Pour la rendre permanente, ajout dans `/etc/network/interfaces` :

```
allow-hotplug enp0s8
iface enp0s8 inet dhcp
```

## 5. Tests de connectivité

- Windows → Debian : `ping 192.168.56.101` : 4/4 réponses, 0 % de perte.
- Debian → Windows : `ping 192.168.56.1` : 100 % de perte au départ, car le pare-feu Windows bloque l'ICMP entrant.
- Correctif, côté Windows (invite de commandes en administrateur) :

```
netsh advfirewall firewall add rule name="Autoriser ping ICMPv4" protocol=icmpv4:8,any dir=in action=allow
```

- Résultat après correctif : [Ping fonctionne de Debian vers windows].

## 6. Premier login et vérifications

Connexion avec l'utilisateur `benk`, puis :

```
hostnamectl
ip a
```

Résultat de `hostnamectl` : [coller la sortie : Static hostname, Operating System, Kernel, Architecture].
Résultat de `ip a` : lo (127.0.0.1), enp0s3 (10.0.2.15/24), enp0s8 (192.168.56.101/24).

## 7. Snapshot

- Snapshot **« base-propre »** créé après installation, configuration réseau et vérifications.
- Il sert de point de retour pour tous les tests suivants.

## 8. Points d'attention

- `sudo` absent : utiliser `su -`, ou l'installer avec `apt install sudo` puis `usermod -aG sudo benk`.
- L'adresse de enp0s8 dépend du DHCP de VirtualBox : elle peut changer, à revérifier avec `ip a`.
