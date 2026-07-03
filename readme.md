## 1. QU'EST-CE QUE PFSENSE ?

### 1.1 Origines de pfSense
pfSense est le firewall open source de référence. 2001 CLI PF dans OpenBSD.

Le pfSense® Project est une distribution gratuite et open soruce qui est une distribution FreeBSD customisée pour permettre un usage en tant que parefeu et routeur géré intégralement par une interface web simple d'usage. Pas 'inquiétude,' aucune connaissance en/de FreeBSD n'est nécessaire pour déployer et utiliser pfSense. La plupart des usagers de pfSenses n'y sotn d'ialleurs pas famiiers et n'ont jamais utilisé FreeBSD en dehors de pfSense. C'est un projet populaire aux millions de téléchargements et aux plusieurs centaines de millers d'installations actives depuis son lancement. Cela va du PC particulier aux réseaux de milliers de périphérques d'universités, de grandes entrprieses et autres orgnasaitons en passant par de petits réseaux domestiques.

C'est un projet qui a vu le jour en 2004, à l'origine c'est un fork du projet m0n0wall. m0n0wall était un proejt de firewall/routeur visant à créer un package logiciel embarqué complet de routeur/firewall qui, lorsqu'u tilisté avec un PC embarqué, emporterait toutes les fonctionnalités majeures de parefeu commericaux, ce qui incluait aussi la facilité d'utilisation mais à une fraction du prix (logiciel gratuit). Basé sur la version bare-bones de FreeBSD m0n0wall embarque un server web, PHP et quelques utilitaires. Toute la ocnfiguration est stockée dans un fichier XML unique afin de garantir la transparence.
Note importante: m0n0wall est possiblement le premier système UNIX qui a sa configuration de boot gérée par PHP à la place des scripts shell habituels, et à avoir otute sa configuration système stockée au format XML.  Sa spécificité était aussi de cibler des ressources hardware limitées. 
A l'origine, pfSense avait pour but de fournir uen solution de firewall/routeur avec un nombre de capacités étendues pour des PC plus puissants et du harware de type serveurs. pfSense a conbtinué dévoluer dans le temps et de founir firewall, VPN, IDN/IPS, et plus de fonctionnalités qui tournent usr du matériel comme de petite PME jusqu'aux fournisseurs de services de plus grande envergure.

"making sense of PF" (PF = ^packet filter technology, au coeur du projet). PF dans FreeBSD fait le meme taff de PF et de QoS et de Firewall que pfSense, mais pfSense lui permet de gérer, monitorer et mainteir beaucoupl plus facilmenet que PF grâce à un GUI facile à prendre en main et services customisé par dessus l'OS et les packaques pertinetnes. Au final solution compelte firewall/routeur/VPN qui est capable deboen plus que la somme de ses composants. Ajotu de valeur vient de cette intégration et de ses propriétés émergentes.

pfSense est un parefeu stateful flexible, une plateforme de routage et comprend un bon nomfre de fonctionnalités. Il gère la restuaration de configuraiton, le NAT, les VLAN, le multi WAN, le wifi, le port forwarding (redirection de ports), le routage, prend en charges les VPN avec IPSec et OpenVPN. Il s'administre via une interface Web dans un portail web captif. ENtre autres feautres il permet uassi le monitoring du parefeu, le logging, l'analyse de traffic, le sniffing, la capture de paquets et la résolution d'incidents (troubleshooitng). Il prend en charge l'installation de logiciels tiers. Le système de gestion de paquets permet des extensions encore plus étendues sans le risque d'ajouter du bloatware et de potentielles vulnérabilités de sécurité à la distribution de base. 

### 1.2 Les origines : distribution customisée de FreeBSD

pfSense est un projet basé sur FreeBSD. LEs développeurs du projet sont des dev FreeBSD.

Définition de pfSense d'après section About de freebsd.org:

"pfSense is a FreeBSD based network security solution. pfSense software, with the help of the package system, is able to provide the same functionality or more of common commercial firewalls, without any of the artificial limitations. It has successfully replaced every big name commercial firewall you can imagine in numerous installations around the world."

FreeBSD vs Linux: FreeBSD, jusqu'à version 13, pas fourni avec environnement de bureau sans installation supplémentaire à partir de sa collection de ports et de paquets. TEmps, efforts, nombreuses instructions écrites pour y parvenir. XFCE4 par exemple (source: article développez.com). BIOS/Micrologiel doivent être à jour. FreeBSD en dekstop fait débat dans les communautés d'utilisateurs. Sembe losuffir sur la prise dnc harge d'applicaitons tierces. FreeBSD a les soucis que Linux a connu il y a 10 ans. Semble plus orienté comme OS serveur et retours de leurs propres dév sur les forums s'oriente dasn ce sens. FreeBSD serveur est paritellement par des neterperises utilsiant FreeBSD en production vs FreeBSD desktop OS de bureu est un effoert entièrement bénévole. Instllation desktop ne semble pas intiutiivé et le support matériel idem. Fiasible pour des initiés, la compétence de l'utilisateur entre en jeu donc pour cet usage Desktop.

| Critère | Linux | FreeBSD |
|----------|-------|----------|
| Architecture | noyau monolitique | micro-noyau modulaire | -> totues les funcs du SE intégrées au noyau principal vs fonctionnalités ajoutées ou supprimées selon les besoins
| Licence |  GNU GPL | licence BSD plus permissive |
| Systèmes de fichiers | ext4 | UFS |
| Communauté | très active | plus petite, très dévouée |
| Support |  |  |
| Performances | plus rapide |  |
| Sécurité |  |  |
| Fonctionnalités |  |  |
| Gestion des paquets | APT | PKG |
| Usage |  serveurs/evts d'entreprise | pare-feux/serveurs Web  |

| Critère | Linux | FreeBSD |
| :--- | :--- | :--- |
| **Architecture** | Noyau monolithique (toutes les fonctions clés sont dans le bloc central du noyau). | Noyau monolithique mais modulaire (très propre, basé sur la pure tradition UNIX). |
| **Licence** | GNU GPL (oblige à redistribuer le code modifié sous la même licence). | Licence BSD (très permissive, permet de modifier le code et de le rendre propriétaire). |
| **Système de fichiers** | Principalement **ext4** (ou Btrfs). | **UFS** par défaut, mais supporte nativement **ZFS** (ultra-robuste pour le stockage et les snapshots). |
| **Gestion des paquets** | APT (Debian/Ubuntu), YUM/DNF (RedHat/Fedora). | **PKG** (paquets binaires) et le système de **Ports** (compiler depuis les sources). |
| **Performances** | Très rapide sur le calcul brut et la virtualisation moderne. | Connu pour sa **pile réseau (TCP/IP) ultra-optimisée** et sa stabilité sous forte charge réseau. |
| **Sécurité** | Excellente, mais dépend de nombreuses couches ajoutées (SELinux, AppArmor). | Excellente de base, code très audité, intègre nativement **PF (Packet Filter)** qui est la référence. |
| **Usage en entreprise** | Serveurs d'application, Cloud, Docker, OS de bureau. | **Infrastructures réseau (routeurs, pare-feux comme pfSense)**, stockage (TrueNAS), serveurs web massifs (Netflix). |



Licence BSD = Berkeley Software Distribution License

FreeBSD est un SE UNIX open-source basé sur la base de code BSD




## 2. POURQUOI UTILISER PFSENSE ?

- Sécurité: Firewalling: PF
- Interconnexion: VPN
- Optimisaiton: Proxy, QoS
- Services réseaux: DNS, DHCP, routage inter VLAN
--
- Open source et communauté très active
- Stable et fiable reconnues
- Gratuit vs concurrents extrêmemtn chers (Cisco, Fortinet)
- Excellente itnégration (cf stabilité) -> pérennité au sei nde l'rognasiiton
- Multifocntion: pas que firewall, plugins complémentaires fournis vers UTM intégrant fonctionnaltiés complémentireas
- Performant: répond au stress testing treès bien, plan de reodnances et de haute disponibiltié
- MOdulabe grace aux plugins pour compléter les foncitonnalités de base
- 

* **Gratuité et absence de limites :** Contrairement aux concurrents extrêmement chers, pfSense est gratuit et ne possède aucune limitation artificielle (pas de licence par nombre de périphériques ou de fonctionnalités).
* **Scalabilité et performance :** Très performant face au stress testing, il s'adapte à la croissance de l'entreprise (scalable). Il intègre aussi des plans de redondance et de haute disponibilité (CARP).
* **Modularité (Vers l'UTM) :** Il est multi-fonctions. Grâce à ses paquets complémentaires (plugins), il évolue d'un simple pare-feu vers un UTM (Unified Threat Management) complet.
* **Stabilité et Pérennité :** Basé sur FreeBSD, sa stabilité et sa fiabilité sont reconnues, ce qui garantit une excellente intégration et une pérennité au sein de l'organisation.
* **Open Source :** Projet transparent avec une communauté très active pour le support et les mises à jour.


## 3. DANS QUEL(S) CONTEXTE UTILISER PFSENSE ?

Différents scnéarios de déploiement:

- petis réseaux d'entprise
- larges réseuax

- FIrewall de perimètre
- Modemou poit nd'accès sans fil
- Routeur
- Switch

=> 2tute prélablave contexte tehcniquet et besoin eaxct.


Le déploiement de pfSense s'adapte à différents scénarios après une étude préalable du contexte technique et des besoins exacts :

* **Segmentation et périmètre :** Utilisé comme firewall de périmètre pour protéger l'accès Internet, ou comme routeur/switch virtuel pour segmenter les réseaux de l'entreprise (ex: isoler les serveurs du LAN).
* **Gestion des accès et filtrage (SquidGuard) :** En entreprise, pour appliquer des politiques de sécurité par groupe ou par plages horaires, complétées par des blacklists pour bloquer les sites indésirables.
* **Traçabilité et Wi-Fi Invités (Portail Captif) :** Dans un contexte d'accueil du public ou d'accès Wi-Fi, pour forcer l'authentification et tracer légalement qui est connecté et depuis où.
* **Flexibilité de taille :** Convient aussi bien aux petits réseaux d'entreprise (PME) qu'aux larges réseaux grâce à sa capacité de virtualisation.


## 4. COMMENT ON S'EN SERT ?

Comprendre son rôle de passerelle. VM doit avoir au moins 2 cartes réseaux: une WAN connectée au réseau extérieur (NAT ou bridge pour récupérer une IP depuis la box internet au même titre que lh'ôte) et une LAN conenctée a switch virtuel (via vSwitch sans carte physique / LAN Segment) pour êter protégées.

## 5. IMPLEMENTATION DANS VMWARE

1 Serveur web dans une DMZ
1 PC client dans une LAN

ETAPE 01: Yéméharger te instlaler et ocnfiguer la VM pfSense

ETAPE 02: au boot, on atribu les IP Des itnerfaces

ETE03/ amdoinsitraiton de pfSense via le GUI par l'IP LAN de pfSense.

Schéma réseau:


# TP – Déploiement et Sécurisation d'un réseau d'entreprise avec pfSense sur VMware

## Objectifs du TP

- Installer et configurer pfSense dans un environnement virtualisé VMware.
- Mettre en place un routage et un filtrage Stateful de base entre un réseau WAN et un réseau LAN.
- Déployer les services réseau essentiels (DHCP, DNS, QoS pour la voix).
- Sécuriser les accès et analyser le trafic via les modules avancés (Portail Captif, SquidGuard, Snort).

## Prérequis / Architecture

- **1 VM pfSense**
  - 2 cartes réseau :
    - WAN
    - LAN
- **1 VM Client** (Windows ou Linux)
  - Située sur le réseau LAN
  - Permet de tester et administrer pfSense

---

# Travail à faire

## Mission 1 — Configuration réseau sous VMware & installation

1. Sur VMware, créer un switch virtuel (ou segment LAN) nommé **LAN-Entreprise**.
2. Configurer la VM **pfSense** :
   - Carte 1 : **Pont** ou **NAT** (WAN)
   - Carte 2 : **LAN-Entreprise** (LAN)
3. Lancer l'installation de pfSense via l'ISO.
4. À la fin de l'installation, configurer les adresses IP via la console :
   - Interface **WAN** : DHCP
   - Interface **LAN** : `192.168.X.254/24` (par exemple)

> 📸 **SCREEN 1 EN ATTENTE**  
> Capture de la console pfSense finale affichant les adresses IP WAN et LAN.

---

## Mission 2 — Configuration des services réseau de base

1. Se connecter sur la VM cliente (placée sur le segment **LAN-Entreprise**).
2. Accéder à l'interface Web de pfSense via l'adresse IP du LAN.
3. Activer et configurer le serveur **DHCP** sur l'interface LAN afin de distribuer des adresses IP au client.
4. Activer le **DNS Resolver** pour permettre la résolution de noms.
5. **Bonus QoS :**
   - Créer une règle de **Traffic Shaper (QoS)** simple afin de prioriser les flux **VoIP** sur le LAN.

---

## Mission 3 — Sécurisation, filtrage et modules avancés

### Firewall

Créer une règle :

- Interdire au LAN de **pinguer le WAN**.
- Autoriser la navigation Web (**HTTP/HTTPS**).

### Proxy (SquidGuard)

- Installer le paquet **SquidGuard**.
- Activer le filtrage d'URL.
- Bloquer une catégorie de sites (ex. : réseaux sociaux).

### Portail Captif

- Activer le portail captif sur le LAN.
- Configurer une authentification par base locale afin de simuler un accès Wi-Fi invité.

### IDS/IPS (Snort)

- Installer le paquet **Snort** sur l'interface WAN.
- Activer les règles de détection de scans de ports.

> 📸 **SCREEN 2 EN ATTENTE**  
> Capture de la page de blocage SquidGuard ou du Portail Captif depuis la VM cliente.


## 6. BUT DE L'OUTIL EN ENTREPRISE


========

Sources:

Projet FreeBSD:

https://www.freebsd.org/
https://www.freebsd.org/projects/newbies/
https://docs.freebsd.org/en/books/handbook/preface/


Documentation officielle pfSense:

Stateful vs Stateless:

https://www.redhat.com/fr/topics/cloud-native-apps/stateful-vs-stateless
https://www.fortinet.com/fr/resources/cyberglossary/stateful-vs-stateless-firewall

Le projet m0n0wall!
https://m0n0.ch/wall/index.php

Qu'est-ce que la QoS

https://www.fortinet.com/fr/resources/cyberglossary/qos-quality-of-service

PF dans FreeBSD

https://docs.freebsd.org/en/books/handbook/firewalls/#firewalls-pf


FreeBSD vs Linux
https://systeme.developpez.com/actu/313566/FreeBSD-peut-il-vraiment-etre-considere-comme-un-systeme-d-exploitation-de-bureau-Si-oui-comme-OS-desktop-pourquoi-choisir-Linux-et-non-FreeBSD/

Comprendre les différences fondamentales entre Linux et FreeBSD
https://goodtech.info/quelles-sont-les-differences-entre-linux-et-freebsd/
https://serverspace.io/fr/about/blog/freebsd-vs-linux-comparison/


QU'est ce que la licence BSD ?

https://appmaster.io/fr/blog/quest-ce-que-la-licence-bsd

Licence GNU GPL v3:

https://www.gnu.org/licenses/gpl-3.0.html


Codebase Berkeley:
https://codebase.studentorg.berkeley.edu/


Formation pfSense
https://www.youtube.com/watch?v=R0BT38ZCY88&list=PL1aYsXmhJ1Wc0cTtvo4b-L9cx4c9NCMo3
https://www.youtube.com/watch?v=Vmd9Amz524U