---
title: "Construction d'un réseau temporaire pour un festival"
date: 2011-07-31T01:54:00+02:00
---

Bonsoir à tous,

Depuis maintenant quatre années, l'association Art en Sort organise un festival de musique à Fécamp. Fesant partie de l'organisation depuis le départ, il m'est souvent arrivé de veiller à la configuration du parc informatique.

L'an passé, nous avons déplacé les bureaux durant la manifestation, nécessitant de déplacer la connexion à Internet. Nous en avons profité pour ouvrir un accés à Internet dans la zone commune des loges... Enfin, nous avons installé une webradio en direct du site du festival, et il a fallu s'assurer que la bande passante nécessaire à la webradio ne soit pas utilisée par les internautes des loges.

La première difficulté est de d'apporter du réseau informatique jusqu'aux loges. Cette zone est dans un batiment distint et il faut faire passer un cable ethernet de 100 mètres de long entre la box de l'asso, le faire passer à l'extérieur et le rentré de nouveau dans le batiment ! Tout un sport !

Pour que le lien fonctionne : il est nécessaire d'avoir des switchs de chaque coté qui sont capables de négocier entre eux la qualité de la communication : on ressent la limitation technique des cables réseaux traditionnels qui se situent à 100 mètres.

La deuxième difficulté est de fabriquer un réseau temporaire, qui est capable de dissocier les flux de la webradio et des loges.Nous allons monter un petit serveur DHCP/DNS/Proxy sur un vieil ordinateur pouir gérer le réseau des loges. Celui de la webradio sera connecté directement à la box. Ce serveur va aussi permettre de mettre en place un filtrage des sites internet : nous fournissons gratuitement du net, nous ne souhaitons pas que des usages illégaux y soient réalisés.

**Programme de ce billet : installer un serveur DHCP/DNS et faire le routage entre le réseau de la boxe et celui des loges.**

Le serveur vient d'être réinstallé de zéro avec un debian 6 à jour. Seul un accès SSH a été installé pour éviter de dupliquer les claviers et les écrans sur mon bureau.

Il faut absolument que le serveur soit équipé de deux cartes réseau : une reliée à la box et l'autre relié au réseau des loges. Dans ma configuration, la carte eth0 est relié aux loges et eth1 relié à la box.  
La configuration de eth1 est en fonction du réseau de la box. Ici : IP = 192.168.0.70/24, gateway 192.168.0.254, dns 8.8.8.8.

Je ne sais pas combien de postes pourraient se relier sur le réseau, mais je préfére voir large. J'ai donc sorti un /16. L'adresse IP de eth0 est  10.1.0.1. Le DHCP pourra attribuer des adresses entre 10.1.0.2 et 10.1.0.254.

Pour s'assurer que la configuration réseau sera effective à chaque redémarrage, nous allons l'entrée en dur dans la configuration.  
Je vous rappele que sur les vieux ordinateurs, je n'utilise pas d'interface graphique : toutes les ressources de la machine sont utilisées pour le traitement du réseau.

Editez le fichier /etc/network/interfaces :

```bash
#Interface reliée vers le réseau des loges  
auto eth0  
iface eth0 inet static  
  address 10.1.0.1  
  netmask 255.255.0.0  
  
#interface reliée vers la box  
auto eth1  
iface eth1 inet static  
  address 192.168.0.70  
  netmask 255.255.255.0  
  gateway 192.168.0.254
```

Pour recharger le réseau avec la nouvelle configuration, vous pouvez faire :

```bash
/etc/init.d/networking restart
```

J'utilise pour ces réseaux temporaires, un petit outils qui comprend DHCP et DNS et qui est assez simple à configurer : dnsmasq. Des paquets sont disponibles pour debian et pour ubuntu.

Via debian, l'installation des paquets est toujours aussi simple :

```bash
apt-get update  
apt-get install dnsmasq
```

Nous allons aussi configurer correctement les DNS. Ceux du serveur seront les DNS transmis à tout le réseau. Pour indiquer les serveurs DNS à utiliser sur le système sous linux, il faut éditer le fichier /etc/resolv.conf :

```
nameserver 8.8.8.8
```

Ici, nous entrons le serveur 8.8.8.8. C'est le serveur DNS de google. Il a l'avantage d'êre très rapide, et d'avoir une adresse reconnaissable de tous. Nous l'utilisons ici pour éviter tout soucis avec les webmails des utilisateurs (on ne sait jamais avec les FAI).

Le gros avantage de dnsmasq, c'est qu'il n'est pas difficile de le configurer et qu'il marche rapidement. Voici les quelques lignes du fichiers de configuration /etc/dnsmasq.conf que j'ai du changer :

```bash

#On active l'envoi d'adresse IP sur le réseau des loges :  
listen-address=10.1.0.1

#On indique la page d'adresses ip qui doivent être attribuées, avec un bail d'une durée de 2h  
dhcp-range=10.1.0.2,10.1.0.254,255.255.0.0,2h

```

On relance Dnsmasq et ... c'est tout !

```bash
/etc/dnsmasq restart
```

Votre réseau est désormais presque fonctionnel : les postes peuvent se brancher et communiquer entre eux. Cependant, cela n'est pas suffisant pour qu'ils aient accès à Internet. Il faudra configurer le serveur pour qu'il fasse du NAT : qu'il enregistre toutes les requêtes du réseau des loges en son nom. Il faudra aussi configurer le proxy pour réduire la bande passante et sécuriser la connexion à internet. Il ne restera plus que la gestion de la bande passante et notre configuration sera prête. Mais tous ces points feront l'objet d'un prochain billet !

A bientôt !