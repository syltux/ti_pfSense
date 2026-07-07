# TP – Déploiement, Segmentation (DMZ) et Sécurisation Réseau avec pfSense sous VMware

## 0. Topologie réseau à réaliser

On va mettre en place un Lab pour la société WashAndClick:

<!-- ```text
mermaid
flowchart LR
    %% Définition du style général
    classDef wan style fill:#f9f,stroke:#333,stroke-width:2px;
    classDef pfsense style fill:#bbf,stroke:#333,stroke-width:3px,stroke-dasharray: 5 5;
    classDef lan style fill:#bfb,stroke:#333,stroke-width:2px;
    classDef dmz style fill:#fbb,stroke:#333,stroke-width:2px;

    %% Zone WAN / Internet
    subgraph WAN_ZONE [Zone Externe - WAN]
        Internet[🌐 Internet / Réseau Box Physique]
    end

    %% Le Coeur de l'infra : pfSense
    subgraph FIREWALL [Périmètre de Sécurité]
        pfsense[🔥 Pare-feu pfSense <br/> FreeBSD / Stateful]
    end

    %% Zone LAN Entreprise
    subgraph LAN_ZONE [Zone Interne - LAN]
        direction TB
        vSwitchLAN[🔌 Segment LAN: 192.168.10.0/24]
        Filtres[🔒 Portail Captif / SquidGuard]
        ClientDebian[💻 Client <br/> IP: DHCP 192.168.10.X]
        
        vSwitchLAN --- Filtres --- ClientDebian
    end

    %% Zone DMZ Serveurs
    subgraph DMZ_ZONE [Zone Isolée - DMZ]
        direction TB
        vSwitchDMZ[🔌 Segment DMZ: 10.0.0.0/24]
        ServerDebian[🖧 Serveur Nginx+Bind9 <br/> IP Statique: 10.0.0.1]
        
        vSwitchDMZ --- ServerDebian
    end

    %% Câblage physique horizontal limpide
    Internet --- |em0: Bridge Pont| pfsense
    pfsense === |em1: Gateway 192.168.10.254| vSwitchLAN
    pfsense === |em2: Gateway 10.0.0.254| vSwitchDMZ

    %% Contrôles logiques (ACLs)
    ClientDebian -.->|Autorisé après Auth| Internet
    Internet -.->|NAT Port Forward 80| ServerDebian
    ServerDebian x-.-x|INTERDIT vers LAN| LAN_ZONE

    %% Application des styles
    class Internet wan;
    class pfsense pfsense;
    class vSwitchLAN,ClientDebian,Filtres lan;
    class vSwitchDMZ,ServerDebian dmz;
``` -->

![Topologie Réseau](/LAB%20WashAndClickv2.svg)

### Plan d'adressage

<!-- | Zone | Interface pfSense | Réseau / Masque | IP pfSense (Passerelle) | Équipement connecté | IP Équipement | Mode d'attribution |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **WAN** | em0 | Dépendant de la Box | IP de la Box | Internet / Réseau Physique | Variable | DHCP (Box) |
| **LAN** | em1 | 192.168.10.0/24 | 192.168.10.254 | Client Debian 12 | Plage DHCP | DHCP (pfSense) |
| **DMZ** | em2 | 10.0.0.0/24 | 10.0.0.254 | Serveur Web + DNS | 10.0.0.10 | Statique / Fixe |

| Zone Réseau | Interface Physique (pfSense) | Type de Connexion / Segment VMware | Adresse Réseau & Masque (CIDR) | IP de la Passerelle (pfSense) | Équipement Virtuel Connecté | IP de l'Équipement | Rôle / Service de l'Équipement | Mode d'Attribution de l'IP | Plage DHCP Active |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **WAN** (Réseau Externe) | `em0` | **VMnet0** (Ponté / Bridged sur le partage de connexion USB) | Dynamique (Fourni par l'opérateur mobile) | IP du Smartphone (ex: `192.168.42.129`) | Pare-feu de périmètre pfSense | Dynamique (ex: `192.168.42.15`) | Accès à Internet et interception des requêtes HTTP entrantes. | **DHCP Client** (via le téléphone) | Aucune (Gérée par le téléphone) |
| **LAN** (Réseau Interne Administrateur) | `em1` | **LAN Segment :** `Segment_LAN` | `192.168.10.0/24` (Classe C Privée) | `192.168.10.254` | **Client-Admin** (Machine Debian 12 avec interface graphique) | Dynamique (ex: `192.168.10.100`) | Poste d'administration, accès à l'interface WebGUI de pfSense, navigation web soumise au Portail Captif et au filtrage SquidGuard. | **DHCP Dynamic** (distribuée par pfSense) | `192.168.10.100` à `192.168.10.199` |
| **DMZ** (Zone Démilitarisée / Isolée) | `em2` | **LAN Segment :** `Segment_DMZ` | `10.0.0.0/24` (Classe A Privée partitionnée) | `10.0.0.254` | **Serveur-Web-DNS** (Machine Debian 12 en ligne de commande) | `10.0.0.10` | Hébergement du serveur HTTP **Nginx** (Site vitrine de WashAndClick) et du serveur **Bind9** (Résolution DNS de la zone). | **Statique / Fixe** (Configuré en dur dans l'OS) | Aucun DHCP (Zone de serveurs uniquement) | -->

| Zone Réseau | Interface Physique (pfSense) | Type de Connexion / Segment VMware | Adresse Réseau & Masque (CIDR) | IP de la Passerelle (pfSense) | Équipement Virtuel Connecté | IP de l'Équipement | Rôle / Service de l'Équipement | Mode d'Attribution de l'IP | Plage DHCP Active |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **WAN** (Réseau Externe) | `em0` | **VMnet0** (Ponté / Bridged sur la clé 4G D-Link) | `192.168.125.0/24` | `192.168.125.1` *(Clé D-Link)* | Pare-feu de périmètre pfSense | Dynamique (ex: `192.168.125.100`) | Accès à Internet et interception des requêtes HTTP entrantes via le NAT. | **DHCP Client** (via la clé 4G) | Aucune (Gérée par la clé 4G) |
| **LAN** (Réseau Interne Administrateur) | `em1` | **LAN Segment :** `Segment_LAN` | `192.168.10.0/24` | `192.168.10.254` | **Client-Admin** (Machine Debian 12 GUI) | Dynamique (ex: `192.168.10.100`) | Poste d'administration, accès à la WebGUI pfSense, Portail Captif et SquidGuard. | **DHCP Server** (distribuée par pfSense) | `192.168.10.100` à `192.168.10.199` |
| **DMZ** (Zone Démilitarisée / Isolée) | `em2` | **LAN Segment :** `Segment_DMZ` | `10.0.0.0/24` | `10.0.0.254` | **Serveur-Web-DNS** (Machine Debian 12 CLI) | `10.0.0.10` | Hébergement du serveur HTTP **Nginx** (Site WashAndClick) et du DNS **Bind9**. | **Statique / Fixe** (Configuré dans l'OS) | Aucun DHCP (Zone serveurs) |

## 1. Objectifs du Travaux Pratiques
À l'issue de ce TP, vous serez capable de :
* Configurer une architecture réseau virtualisée d'entreprise à 3 zones (WAN / LAN / DMZ) sous VMware.
* Installer et réaliser l'initialisation système du pare-feu Stateful pfSense (basé sur FreeBSD).
* Déployer les services réseau critiques (DHCP, DNS, Routage inter-zones).
* Sécuriser les flux de production en appliquant le principe du moindre privilège (ACL/Règles de Firewall).
* Mettre en œuvre des briques de sécurité avancées (Filtrage Web SquidGuard, Traçabilité Portail Captif sur le LAN, Détection d'intrusion Snort).

## 2. Architecture & Prérequis Matériels
Le lab virtuel est entièrement hébergé sur VMware Workstation et se compose de 3 machines :

1. **VM pfSense (Routeur/Firewall de périmètre) :** 
   * OS : FreeBSD 64-bit (ISO pfSense Community Edition)
   * RAM : 2 Go | Disque : 20 Go
   * Carte 1 (WAN) : Mode **Bridged (Pont)** -> Connectée directement à votre box internet (IP publique/physique).
   * Carte 2 (LAN) : Mode **LAN Segment** -> `LAN-Entreprise`
   * Carte 3 (OPT1/DMZ) : Mode **LAN Segment** -> `DMZ-Serveurs`
2. **VM Client (Poste salarié / Administration) :**
   * OS : Debian 12.14.0 (avec environnement de bureau) ou Windows 10/11 Client, au choix.
   * Carte réseau : Mode **LAN Segment** -> `LAN-Entreprise` (Configuration IP : DHCP).
3. **VM Serveur (Serveur Web & DNS en DMZ) :**
   * OS : Debian 12.14.0 CLI (Légère)
   * Carte réseau : Mode **LAN Segment** -> `DMZ-Serveurs` (Configuration IP : Statique).

---

## 3. Travail à réaliser (Pas-à-pas)

### Mission 1 : Préparation de l'infrastructure VMware & Installation de pfSense

0. Pour récupérer l'ISO de pfSense, allez sur https://pfsense.org/
!["Site de pfSense"](/medias/Capture%20d’écran%20(7212).png)
Rubrique Download:
!["Site de pfSense"](/medias/Capture%20d’écran%20(7213).png)
!["Site de pfSense"](/medias/Capture%20d’écran%20(7215).png)
Sélectionnez la version ISO/Virtual Machine:
!["Site de pfSense"](/medias/Capture%20d’écran%20(7216).png)
A ce stade, si vous n'avez pas de compte Netgate vous demandera d'en créer un. Une adresse mail, un mot de passe et un nom d'utilisateur plus tard, vous arrivez sur le récapitulatif du panier:
!["Site de pfSense"](/medias/Capture%20d’écran%20(7220).png)
Complétez le processus de commande, aucun moyen de paiement n'est nécessaire mais vous devrez remplir les coordonnées. Validez et vous obtenez le lien de téléchargement sur la gauche de l'écran. Si vous fermez la page, il vous sera également envoyé par mail.
![alt text](image-7.png)

![alt text](image.png)
![alt text](image-1.png)
On passe la VM en Bridge, on passe le path de l'ISO et on ajuste la mémoire à 2Go:
![alt text](image-2.png)
1. Dans VMware, accédez au gestionnaire réseau virtuel (*Virtual Network Editor*) et créez deux segments LAN distincts : `LAN-Entreprise` et `DMZ-Serveurs`.
![alt text](image-3.png)
![alt text](image-9.png)
![alt text](image-10.png)
![alt text](image-4.png)

Ajouter deux autres cartes réseau :
![alt text](image-8.png)
![alt text](image-11.png)
![alt text](image-12.png)

Enfin bien penser à aller dans Editeur de Réseau et vérifier que Vmnet0 est bien en bridge et sur la bonne carte réseau:

![alt text](image-15.png)

2. Configurez le matériel des trois machines virtuelles conformément aux prérequis ci-dessus (laissez-les éteintes pour le moment).
3. Démarrez la VM pfSense sur l'ISO officielle. Suivez l'assistant d'installation textuel en conservant les options par défaut (système de fichier Auto UFS).
![alt text](image-5.png)
Cliquer sur Accept:
![alt text](image-6.png)
Cliquer sur Install:
![alt text](image-13.png)
![alt text](image-14.png)
![alt text](image-16.png)
![alt text](image-17.png)
![alt text](image-18.png)
![alt text](image-19.png)
![alt text](image-20.png)
![alt text](image-21.png)
![alt text](image-22.png)
![alt text](image-23.png)
![alt text](image-24.png)
![alt text](image-25.png)
![alt text](image-26.png)
![alt text](image-27.png)
![alt text](image-28.png)

On laisse sans redondance car on installe danse une VM, mais le RAID serait pertinent sur une machine physique pour sécuriser le système d'exploitation:
![alt text](image-29.png)
![alt text](image-30.png)
![alt text](image-31.png)
![alt text](image-32.png)
![alt text](image-33.png)
![alt text](image-34.png)
![alt text](image-35.png)
![alt text](image-36.png)
![alt text](image-37.png)
![alt text](image-38.png)
![alt text](image-39.png)
![alt text](image-40.png)
![alt text](image-41.png)
![alt text](image-42.png)
![alt text](image-43.png)
![alt text](image-44.png)
![alt text](image-45.png)
![alt text](image-46.png)
![alt text](image-47.png)
![alt text](image-48.png)
![alt text](image-49.png)
![alt text](image-50.png)
![alt text](image-51.png)
![alt text](image-52.png)
![alt text](image-53.png)
![alt text](image-54.png)
![alt text](image-55.png)
![alt text](image-56.png)
![alt text](image-57.png)
4. Au redémarrage, configurez l'attribution des interfaces sur la console noire :
![alt text](image-58.png)
   * Ne configurez pas de VLANs immédiatement (`n`).
   * Assignez l'interface WAN : `em0` (votre première carte réseau).
   * Assignez l'interface LAN : `em1` (votre deuxième carte réseau).
   * Assignez l'interface optionnelle OPT1 : `em2` (votre troisième carte réseau).
   ![alt text](image-59.png)
   ![alt text](image-60.png)
   ![alt text](image-61.png)
   ![alt text](image-62.png)
   ![alt text](image-63.png)
   ![alt text](image-64.png)
   ![alt text](image-65.png)
   ![alt text](image-66.png)
   ![alt text](image-67.png)
   ![alt text](image-68.png)
   ![alt text](image-69.png)
5. Depuis le menu principal (16 options), sélectionnez l'**Option 2 (Set interface(s) IP address)** pour configurer le LAN :
   * Sélectionnez l'interface LAN.
   * Adresse IP : `192.168.10.254`
   * Masque (Notation CIDR) : `24` (équivalent à `255.255.255.0`).
   * Activer le serveur DHCP sur le LAN : `n`(Non)
   * Plage d'adresses DHCP : `192.168.10.10` à `192.168.10.100`.
   * Protocol WebConfigurator : Répondez `n` (Non) pour conserver le protocole sécurisé HTTPS.

> **📸 CAPTURE ÉCRAN 1 :** Écran de la console pfSense finale affichant le menu principal avec les trois interfaces validées et leurs adresses IP respectives (WAN avec l'IP de votre box, LAN en `192.168.10.254/24`).

---

### Mission 2 : Configuration du Serveur DMZ & Prise de contrôle du Client

0. Installation et configuration de la VM Serveur Web:

![alt text](image-105.png)
![alt text](image-106.png)
![alt text](image-107.png)
![alt text](image-108.png)
![alt text](image-109.png)

1. Démarrez la VM **Serveur (DMZ)**. Configurez son adressage réseau statique dans le fichier `/etc/network/interfaces` :
   * IP : `10.0.0.10` | Masque : `255.255.255.0` | Passerelle : `10.0.0.254` (IP finale de la patte DMZ de pfSense).
2. Installez les services de production de la DMZ :
   ```bash
   apt update && apt install nginx bind9 -y
3. (Optionnel) Ajouter un index.html dans /var/www/html/index.html et l'éditer avec nano.

Démarrez la VM Client (LAN). Vérifiez qu'elle reçoit automatiquement une adresse IP du type `192.168.10.X` via le DHCP du pare-feu.

![alt text](image-70.png)
![alt text](image-71.png)
![alt text](image-72.png)
![alt text](image-73.png)
Le certificat SSL est auto-signé, donc le navigateur lève une alerte:
![alt text](image-74.png)
![alt text](image-75.png)
![alt text](image-76.png)

Ouvrez un navigateur Web sur le client et connectez-vous au panneau d'administration via :

```text
https://192.168.10.254
```
```

> Identifiants d'usine : **admin / pfsense**

![alt text](image-77.png)
![alt text](image-78.png)
![alt text](image-79.png)
![alt text](image-80.png)
![alt text](image-81.png)
![alt text](image-83.png)
![alt text](image-84.png)
![alt text](image-85.png)
![alt text](image-86.png)
![alt text](image-87.png)
![alt text](image-88.png)

Suivez l'assistant **Setup Wizard** initial, configurez le fuseau horaire et modifiez obligatoirement le mot de passe administrateur par défaut si ce n'est pas déjà fait. Idéalement: créer un compte admin (avec un nom différent) et supprimez le compte admin.

![alt text](image-89.png)
![alt text](image-90.png)
![alt text](image-91.png)
![alt text](image-92.png)
![alt text](image-93.png)
![alt text](image-94.png)
![alt text](image-95.png)
![alt text](image-96.png)
![alt text](image-97.png)

![alt text](image-82.png)

Allez dans **Interfaces > OPT1**, cochez la case **Enable Interface**, renommez la zone (**Description**) en **DMZ**.

![alt text](image-98.png)
![alt text](image-99.png)
![alt text](image-100.png)
![alt text](image-101.png)
![alt text](image-102.png)

Attribuez-lui une IPv4 de type **Static IPv4**, renseignez l'adresse IP **10.0.0.254** avec un masque **/24**, puis sauvegardez et appliquez les changements.

# Mission 3 : Durcissement du Pare-feu (Règles d'ACL)

Par défaut, pfSense autorise tout le trafic sortant du LAN, mais bloque l'intégralité du trafic initié depuis ou vers la DMZ.

Nous allons appliquer les exigences de sécurité de l'entreprise.

## Publication du serveur Web (NAT / Port Forwarding)

Allez dans :

**Firewall > NAT > Port Forward**

![alt text](image-103.png)
Avant le hardening, le NAT de pfSense est configuré en auto (il a créé des règles de NAT):
![alt text](image-104.png)

Ajoutez une règle pour rediriger le trafic HTTP arrivant sur l'IP WAN de pfSense (port **80**) vers l'adresse IP privée de votre serveur Nginx en DMZ :

```text
10.0.0.10
```
![alt text](image-118.png)

![alt text](image-119.png)
![alt text](image-120.png)

NB: Ce setup est à activer en production mais pas en dev pour éviter le lockout:

![alt text](image-121.png)

## Isolation stricte DMZ → LAN

### Création d'une première règle interdisant toute le traffic depuis la LAN vers la DMZ sur tous les protocoles

Allez dans :

**Firewall > Rules > DMZ**
![alt text](image-110.png)
Ajoutez une règle d'interdiction explicite (**Block**) :

| Champ | Valeur |
|--------|---------|
| Protocol | Any |
| Source | DMZ net |
| Destination | LAN net |

![alt text](image-111.png)

Cliquer sur "Apply changes":
![alt text](image-112.png)
pfSense confirme la bonne prise en compte de la règle:
![alt text](image-113.png)

### Création d'une seconde règle pour autoriser le traffic sur le port 80 depuis la LAN vers le site Web (pas toute la DMZ)

![alt text](image-114.png)
![alt text](image-115.png)
![alt text](image-116.png)

NB: L'ordre des règles est important: 

![alt text](image-117.png)

> **Objectif :** si le serveur Web est compromis par un pirate, celui-ci ne doit pas pouvoir rebondir sur les postes des salariés.

## Durcissement du LAN (Anti-Ping)

Dans :

**Firewall > Rules > LAN**

Ajoutez une règle bloquant (**Block**) le protocole **ICMP (Ping)** depuis le **LAN** vers l'interface **WAN**, tout en laissant la règle par défaut :

> **Default allow LAN to any**

active en dessous afin de conserver l'accès à Internet.

### 📸 Capture d'écran 2

Vue d'ensemble de la table des règles de filtrage :

```text
Firewall > Rules > DMZ
```

Montrant la règle d'isolation hermétique interdisant à la DMZ de requêter le réseau LAN.

# Mission 4 : Activation des modules de sécurité avancée

## Étape A : Filtrage Web applicatif (Squid & SquidGuard)

Allez dans :

**System > Package Manager > Available Packages**

Recherchez puis installez les paquets suivants :

- squid (Proxy)
- squidGuard (Moteur de filtrage d'URL)

Ensuite :

**Services > Squid Proxy**

- Cochez **Enable Squid Proxy** sur l'interface LAN.
- Activez **Transparent HTTP Proxy** afin d'intercepter automatiquement le trafic HTTP des utilisateurs.

Puis :

**Services > SquidGuard**

- Activez le service.
- Téléchargez une **Blacklist** d'URL.
- Dans l'onglet **Common ACL**, configurez :
  - le blocage des catégories demandées (ex. réseaux sociaux, streaming),
  - ou des plages horaires d'interdiction.

---

## Étape B : Traçabilité et contrôle d'accès (Portail Captif)

Dans cette section, nous allons faire en sorte que tout utilisateur de la LAN qui tente d'accéder à Internet soit obligatoirement logué afin de contrôler et tracer les accès:

Cette section s'appuie sur la vidéo suivante: https://www.youtube.com/watch?v=JmCadrWt1ag

Allez dans :

**Services > Captive Portal**

- Cliquez sur **Add**.
- Nommez la zone :

```text
Acces_Securise_LAN
```

- Validez.

Ensuite :

- Cochez **Enable Captive Portal**.
- Sélectionnez l'interface **LAN**.

Dans la section **Authentication** :

- Choisissez **Local User Manager**. (on peut choisir de se baser sur LDAP ou RADIUS par exemple sur un campus universitaire. Ici on choisit un serveur d'auth local).

Puis allez dans :

**System > User Manager**

Créez un utilisateur de test :

| Paramètre | Valeur |
|-----------|---------|
| Identifiant | salarie |
| Mot de passe | Tssr2026! |

---

## Étape C : Détection d'intrusions réseau (Snort IDS/IPS)

Installez le paquet **Snort** depuis le **Package Manager**.

Puis :

**Services > Snort**

Dans l'onglet **Snort Interfaces** :

- Ajoutez une interface de surveillance liée au **WAN**.

Activez les jeux de règles de détection (signatures de scans de ports agressifs ou de requêtes malveillantes) afin de journaliser les attaques externes directement dans l'interface de monitoring.

# 4. Livrables & Procédure de validation des tests

Pour valider le bon fonctionnement de la maquette, réalisez les tests suivants.

## Validation du portail captif

Depuis la VM cliente LAN :

1. Ouvrez un navigateur.
2. Essayez d'accéder à un site Web externe.
3. Le portail captif pfSense doit intercepter la connexion.
4. Authentifiez-vous avec le compte créé.
5. Vérifiez que l'accès Internet est ensuite autorisé.

---

## Validation de la DMZ (Publication Nginx)

Depuis le client LAN :

```text
http://10.0.0.1
```

Vérifiez l'accès interne au serveur.

Depuis une machine située sur le WAN (ou l'hôte) :

Saisissez l'adresse IP WAN de pfSense dans un navigateur afin de vérifier la redirection NAT vers la page d'accueil Nginx.

---

## Validation du Proxy (SquidGuard)

Une fois authentifié sur le portail captif :

Essayez d'accéder à un site interdit (exemple : Facebook).

La page de blocage SquidGuard doit s'afficher.

### 📸 Capture d'écran 3

Capture du navigateur de la VM cliente LAN montrant la page d'authentification du portail captif exigeant les identifiants locaux avant d'autoriser l'accès au WAN.

# 5. Barème de notation (sur 20 points)

| Critère | Points |
|----------|--------|
| Architecture VMware & adressage des 3 zones (WAN/LAN/DMZ) | **/4** |
| Initialisation de pfSense et services réseau de base (DHCP/Interfaces) | **/4** |
| Mise en œuvre des ACLs et sécurité réseau (NAT, isolation DMZ, blocage ICMP) | **/4** |
| Déploiement et fonctionnement des modules avancés (SquidGuard, Portail Captif, Snort) | **/5** |
| Qualité des tests de validation et pertinence des captures d'écran | **/3** |
