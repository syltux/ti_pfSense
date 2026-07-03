## 1. QU'EST-CE QUE PFSENSE ?

### 1.1 Origines de pfSense

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

Licence BSD = Berkeley Software Distribution License

FreeBSD est un SE UNIX open-source basé sur la base de code BSD




## 2. POURQUOI UTILISER PFSENSE ?

## 3. DANS QUEL(S) CONTEXTE UTILISER PFSENSE ?

## 4. COMMENT ON S'EN SERT ?

## 5. IMPLEMENTATION DANS VMWARE

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




