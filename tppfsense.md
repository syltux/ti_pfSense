# TP – Déploiement, Segmentation (DMZ) et Sécurisation Réseau avec pfSense sous VMware

## 0. Topologie réseau à réaliser

graph TD
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
        pfsense[🔥 Pare-feu pfSense <br/> FreeBSD 12 / Stateful]
    end

    %% Zone LAN Entreprise
    subgraph LAN_ZONE [Zone Interne - LAN]
        direction TB
        vSwitchLAN[🔌 Segment: LAN-Entreprise]
        ClientDebian[💻 Client Debian 12 <br/> IP: DHCP 192.168.10.X]
        PortailCaptif[🔒 Filtrage: Portail Captif <br/> & Proxy SquidGuard]
    end

    %% Zone DMZ Serveurs
    subgraph DMZ_ZONE [Zone Isolée - DMZ]
        direction TB
        vSwitchDMZ[🔌 Segment: DMZ-Serveurs]
        ServerDebian[🖧 Serveur Debian 12 CLI <br/> Nginx & Bind9 <br/> IP Statique: 10.0.0.1]
    end

    %% Interconnexions et Flux Network
    Internet <=>|em0: Bridge Pont| pfsense
    pfsense <=>|em1: Gateway 192.168.10.254| vSwitchLAN
    vSwitchLAN --- PortailCaptif
    PortailCaptif --- ClientDebian
    
    pfsense <=>|em2: Gateway 10.0.0.254| vSwitchDMZ
    vSwitchDMZ --- ServerDebian

    %% Liens logiques de sécurité (ACLs)
    ClientDebian -.->|1. Autorisé après Auth Web| Internet
    Internet -.->|2. NAT Port Forward 80| ServerDebian
    ServerDebian -.-x|3. INTERDIT explicitement| LAN_ZONE

    %% Application des styles
    class Internet wan;
    class pfsense pfsense;
    class vSwitchLAN,ClientDebian,PortailCaptif lan;
    class vSwitchDMZ,ServerDebian dmz;

## 1. Objectifs du Travaux Pratiques
À l'issue de ce TP, l'étudiant sera capable de :
* Configurer une architecture réseau virtualisée d'entreprise à 3 zones (WAN / LAN / DMZ) sous VMware.
* Installer et réaliser l'initialisation système du pare-feu Stateful pfSense (basé sur FreeBSD).
* Déployer les services réseau critiques (DHCP, DNS, Routage inter-zones).
* Sécuriser les flux de production en appliquant le principe du moindre privilège (ACL/Règles de Firewall).
* Mettre en œuvre des briques de sécurité avancées (Filtrage Web SquidGuard, Traçabilité Portail Captif sur le LAN, Détection d'intrusion Snort).

## 2. Architecture & Prérequis Matériels
Le lab virtuel est entièrement hébergé sur VMware Workstation et se compose de 3 machines :

1. **VM pfSense (Routeur/Firewall de périmètre) :** * OS : FreeBSD 64-bit (ISO pfSense Community Edition)
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

0. 
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

Démarrez la VM Client (LAN). Vérifiez qu'elle reçoit automatiquement une adresse IP du type `192.168.10.X` via le DHCP du pare-feu.

Ouvrez un navigateur Web sur le client et connectez-vous au panneau d'administration via :

```text
https://192.168.10.254
```

> Identifiants d'usine : **admin / pfsense**

Suivez l'assistant **Setup Wizard** initial, configurez le fuseau horaire et modifiez obligatoirement le mot de passe administrateur par défaut.

Allez dans **Interfaces > OPT1**, cochez la case **Enable Interface**, renommez la zone (**Description**) en **DMZ**.

Attribuez-lui une IPv4 de type **Static IPv4**, renseignez l'adresse IP **10.0.0.254** avec un masque **/24**, puis sauvegardez et appliquez les changements.

# Mission 3 : Durcissement du Pare-feu (Règles d'ACL)

Par défaut, pfSense autorise tout le trafic sortant du LAN, mais bloque l'intégralité du trafic initié depuis ou vers la DMZ.

Nous allons appliquer les exigences de sécurité de l'entreprise.

## Publication du serveur Web (NAT / Port Forwarding)

Allez dans :

**Firewall > NAT > Port Forward**

Ajoutez une règle pour rediriger le trafic HTTP arrivant sur l'IP WAN de pfSense (port **80**) vers l'adresse IP privée de votre serveur Nginx en DMZ :

```text
10.0.0.1
```

## Isolation stricte DMZ → LAN

Allez dans :

**Firewall > Rules > DMZ**

Ajoutez une règle d'interdiction explicite (**Block**) :

| Champ | Valeur |
|--------|---------|
| Protocol | Any |
| Source | DMZ net |
| Destination | LAN net |

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
