# connect_homelab_to-_internet
A very cheap way to connect your physical home lab to ISP network without investing in expensive equipments


# Lab : Configuration de ma topology réseau personnelle à domicile pour séparer mes LABs de mon réseau principal 

## Objectif
Créer une topologie à moindre coût pour les labs et pouvant joindre internet avec 2 switches (mais un switch doit etre de niveau trois malheureusement si on veut faire du routage entre VLANs) une ancienne machine peu couteuse qui va nous servir pour le nat entre notre réseau LAB et notre réseau HOME et le routeur du fournisseur d'accès internet (donc pas besoin de faire l'achat ici puisqu'il est déja présent), des VLANs (ils peuvent être ajustés comme on le souhaite c'est juste que la topologie que j'utilise utilise 3 VLANs donc 10, 20 et 30 qui est le management et le VLAN natif), configurer le trunking pour le routage des VLANS à l'intérieur de la partie LAB, et tester la communication entre postes. En bonus configuration d'un controlleur WLAN mais que je n'ai pas implémenté physiquement car je n'ai pas de wireless LAN controlleur physique je le fais juste dans le lab. 

## Technologies utilisées
- Machine Linux (Debian 12) dans mon cas
- Cisco Packet Tracer
- VLAN, trunk, NAT, SVI, access-list, serveur dhcp, DNS
- inter-VLAN routing (layer 3 switch) si on a un routeur on peut le faire le "router on a stick" et éliminer le switch
- CLI Cisco IOS
- protocole routage OSPF (mais on pourrait bien utiliser l'IGP que l'on souhaite EIGRP OU RIP)

## Topologie
voir l'image de la topologie dans les fichiers

## Étapes
1. Création de la topologie réseau
Le routeur du fournisseur d'accès internet ne fourni pas une interface en ligne de commande d'ailleurs le seul paramétrage à faire à ce niveau c'est la plage d'adresse IP pour les appareil dans le LAN de la partie HOME si on le veut. Mais la fonctionnalité dhcp étant déja activé on peut la laisser ainsi.
dans le lab notre ISP est représenté par le serveur ISP qui dispose d'un serveur web et d'un serveur DNS configuré pour tester la connectivité vers le WAN. (nom du site frankndounga.com). j'ai configurer la partie WAN du routeur du FAI en static vous pouvez le faire en dynamic si vous voulez pour mieux refléter ce qui se fait dans le monde réel.

2. Attribuer deux interfaces réseau à la machine Linux (ici routeur test)
une interface sera dans la partie réseau de notre HOME 192.168.0.0/24
une autre dans la partie réseau de notre LAB 172.16.0.0/30 (on a besoin que de deux adresses d'où le masque en /30) c'est la connexion entre notre switch de niveau 3 et notre machine Linux (routeur)
NB: puisque ceci est la topologie sur le simulateur nous allons tester la connectivité avec un routeur et non avec une machine Linux comme nous le faisons en production. Configuration de la passerelle par défaut qui est l'adresse interne du routeur du ISP sur le routeur test soit (192.168.0.1)

3. configurer le NAT sur le routeur test pour permettre aux paquets sortant de la zone lab de pourvoir sortir du réseau LAB vers le WAN par le routeur test.

4. activer le protocole ospf sur l'interface interne du au LAB sur le routeur et sur le switch pour les deux interfaces pour que le routeur test puisse connaitre toutes les routes du LAB. En situation réelle ce sera les routes statiques puisque on va utiliser un serveur Linux.

5. activer le routage sur le switch multilayer (ip routing) ainsi que la route par défaut pointant vers l'interface du routeur (ou bien activer la determination de la route par défaut par le protocole ospf mais en situation réelle ce ne sera pas le routeur donc pas vraiment adapté si on ne s'y connais pas)

6. Je teste la connectivité entre le multilayer switch et le routeur test ainsi qu'avec l'extérieur. Selon moi c'est la connexion la plus importante de notre topology ici.

7. Je configure notre home LAB comme on le désire maintenant avec le serveur dhcp au niveau du switch du niveau trois pour chaque vlan. nous allons donc configurer les SVI pour chaque vlan de notre lab. mettre la liaison entre le switch du lab et le switch de niveau trois en trunk. nous avons déja activer le routage auparavant sur le switch de niveau 3. La config du lab ici n'est pas trop importante parceque cela dépendra des fonctionnalités que l'on veut travailler dans le lab. L'important est que toutes les machines du LAB puissent accéder à internet via les configurations que nous avons faite.

8. En testant ici la connectivité on se rend compte que l'access list (1) que nous avions configurer pour le NAT n'est pas tout à fait correcte il faut la modifier pour ajouter les possibilités de translations pour chaque réseau du LAB. Actuellement elle ne prend que la translation du lien avec le switch de niveau 3.
concrètement un exemple serait "access-list 1 permit 192.168.10.0 0.0.0.255" pour notre VLAN 10 par exemple.

9. Test de connectivité entre les VLANs

## Résultats
- Ping réussi entre les hôtes du lab vers internet


## Fichiers inclus
config routeur test
config multilayer switch
topologie réseau
fichier packet tracer

## Problèmes rencontrés
Le véritable problème rencontré a été de comprendre qu'il fallait aussi la translation des adresses du réseau du LAB par le routeur et pas seulement celle du multilayer switch. La solution c'est juste d'ajouter des ACE dans l'accès list configurer sur le routeur pour les réseaux internes du lab (différents VLAN)

## Améliorations possibles
Les améliorations sont tellement multiples que je ne saurais où commencer mais je pourrais citer l'implémentation du DNS surtout que en production j'utiliserai un serveur Linux, s'il y a un autre site faire la connexion entre les deux sites.

## Ce que j'ai appris
Comment mettre en pratique quelques éléments de mon CCNA

## Comment explorer le lab
packet tracer version 8.2.2
lire le README et ouvrir le fichier.pkt
