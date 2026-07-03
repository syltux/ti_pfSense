# TP – Déploiement, Segmentation (DMZ) et Sécurisation Réseau avec pfSense sous VMware

## 1. Objectifs du Travaux Pratiques
À l'issue de ce TP, l'étudiant sera capable de :
* Configurer une architecture réseau virtualisée d'entreprise à 3 zones (WAN / LAN / DMZ) sous VMware.
* Installer et réaliser l'initialisation système du pare-feu Stateful pfSense (basé sur FreeBSD).
* Déployer les services réseau critiques (DHCP, DNS, Routage inter-zones).
* Sécuriser les flux de production en appliquant le principe du moindre privilège (ACL/Règles de Firewall).
* Mettre en œuvre des briques de sécurité avancées (Filtrage Web SquidGuard, Traçabilité Portail Captif sur le LAN, Détection d'intrusion Snort).

## 2. Architecture & Prérequis Matériels
Le lab virtuel est entièrement hébergé sur VMware Workstation / ESXi et se compose de 3 machines :

1. **VM pfSense (Routeur/Firewall de périmètre) :** * OS : FreeBSD 64-bit (ISO pfSense Community Edition)
   * RAM : 1 Go | Disque : 20 Go
   * Carte 1 (WAN) : Mode **Bridged (Pont)** -> Connectée directement à votre box internet (IP publique/physique).
   * Carte 2 (LAN) : Mode **LAN Segment** -> `LAN-Entreprise`
   * Carte 3 (OPT1/DMZ) : Mode **LAN Segment** -> `DMZ-Serveurs`
2. **VM Client (Poste salarié / Administration) :**
   * OS : Debian 12 (avec environnement de bureau) ou Windows Client.
   * Carte réseau : Mode **LAN Segment** -> `LAN-Entreprise` (Configuration IP : DHCP).
3. **VM Serveur (Serveur Web & DNS en DMZ) :**
   * OS : Debian 12 CLI (Légère - *from scratch*)
   * Carte réseau : Mode **LAN Segment** -> `DMZ-Serveurs` (Configuration IP : Statique).

---

## 3. Travail à Réaliser (Pas-à-pas)

### Mission 1 : Préparation de l'infrastructure VMware & Installation de pfSense
1. Dans VMware, accédez au gestionnaire réseau virtuel (*Virtual Network Editor*) et créez deux segments LAN distincts : `LAN-Entreprise` et `DMZ-Serveurs`.
2. Configurez le matériel des trois machines virtuelles conformément aux prérequis ci-dessus (laissez-les éteintes pour le moment).
3. Démarrez la VM pfSense sur l'ISO officielle. Suivez l'assistant d'installation textuel en conservant les options par défaut (système de fichier Auto UFS).
4. Au redémarrage, configurez l'attribution des interfaces sur la console noire :
   * Ne configurez pas de VLANs immédiatement (`n`).
   * Assignez l'interface WAN : `em0` (votre première carte réseau).
   * Assignez l'interface LAN : `em1` (votre deuxième carte réseau).
   * Assignez l'interface optionnelle OPT1 : `em2` (votre troisième carte réseau).
5. Depuis le menu principal (16 options), sélectionnez l'**Option 2 (Set interface(s) IP address)** pour configurer le LAN :
   * Sélectionnez l'interface LAN.
   * Adresse IP : `192.168.10.254`
   * Masque (Notation CIDR) : `24` (équivalent à `255.255.255.0`).
   * Activer le serveur DHCP sur le LAN : `y` (Oui).
   * Plage d'adresses DHCP : `192.168.10.10` à `192.168.10.100`.
   * Protocol WebConfigurator : Répondez `n` (Non) pour conserver le protocole sécurisé HTTPS.

> **📸 CAPTURE ÉCRAN 1 :** Écran de la console pfSense finale affichant le menu principal avec les trois interfaces validées et leurs adresses IP respectives (WAN avec l'IP de votre box, LAN en `192.168.10.254/24`).

---

### Mission 2 : Configuration du Serveur DMZ & Prise de contrôle du Client
1. Démarrez la VM **Serveur (DMZ)**. Configurez son adressage réseau statique dans le fichier `/etc/network/interfaces` :
   * IP : `10.0.0.1` | Masque : `255.255.255.0` | Passerelle : `10.0.0.254` (IP finale de la patte DMZ de pfSense).
2. Installez les services de production de la DMZ :
   ```bash
   apt update && apt install nginx bind9 -y