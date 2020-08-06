---
title: "Faire revivre une raie manta"
date: 2012-08-05T22:18:48+02:00
draft: false
---

Suite à des problèmes de connexion à Internet (perte de synchronisation ADSL avec ma freebox revolution), j'ai recherché une solution pour avoir un peu de connexion.

![Photo de raie](/img/raiemanta.jpg)

Regardant sur de nombreux forums, il m'a souvent été signifié qu'il s'agissait d'un problème de surchauffe de freebox. Pour vérifier, j'ai décidé de ressortir l'ancien modem ADSL de mes parents, une célèbre raie manta de l'époque de Wanadoo, un alcatel speedtouch USB, plus connu sous le nom de SpeedTouch 330.

Premières recherches :
======================

Tout d'abord, j'ai la chance d'être chez un hébergeur qui permet la connexion d'autres modems que son propre routeur ADSL (c'est à dire la freebox révolution). Ainsi, si vous le souhaitez, vous pouvez brancher n'importe quel routeur ADSL, à la condition qu'il puisse être configuré comme indiqué sur cette page : http://www.free.fr/assistance/2253-avec-un-modem-adsl.html.


Des drivers sous linux permettent cette configuration avec la raie manta. Sous windows, la configuration est assez limité avec les drivers par défaut. Je n'ai pas effectué de recherches pour Mac OS. Il semble cependant qu'il y avait un portage du drivers de linux jusque Mac Os 10.4. A tester ...

Cette configuration fonctionne sous Debian 6.

Installation :
==============

Avertissement : L'installation de ce modem rend votre ordinateur vulnérable. En effet, ce dernier est directement accessible depuis l'adresse IP offerte par Free. Attention donc à ne pas connecter un PC dont la sécurité est désactivé voir inexistante.

Pré-requis : Vous aurez besoin de votre adresse IP ainsi que celle de la gateway de Free. Ces informations sont disponibles dans la rubrique "Ma freebox" de l'interface d'administration de Free.
Etape 1 : Installation des outils

Tout d'abord, il faut télécharger le firmware du modem. Ce dernier est disponible à l'adresse ftp://ftp.linux.it/pub/People/md/warez/speedtouch-firmware_0.3012k_all.deb. Une copie est aussi disponible [ici](/download/speedtouch-firmware_0.3012k_all.deb)

Il faut ensuite installer les outils pour que votre linux puisse effectuer des communications en ATM :

```bash
apt-get install atm-tools
```

Etape 2 : Configuration des outils
==================================

Copier le fichier de configuration suivant :

```bash
cp /usr/share/doc/ppp/examples/peers-pppoe /etc/ppp/peers/adslscript
```

Puis éditez le.

```bash
nano /etc/ppp/peers/adslscript
```

Commentez les lignes user

```bash
# user "myusername@myprovider.com"
#eth0
#plugin rp-pppoe.so
```

Puis remplacez ces lignes par :

```bash
plugin pppoatm.so
8.36
```

Etape 3 : chargement du logiciel et connexion
=============================================

Pour charger le module dans le noyau, lancez la commande

```bash
modprobe speedtch
```

Afin d'observer le bon lancement du modem, tapez la commande suivante qui permet de suivre l'évolution l'un des fichiers de log du noyau :

Lancez tail -f /var/log/messages

Si votre modem, votre connexion et votre configuration fonctionnent, vous devirez voir apparaître :

```bash
ATM dev 0: ADSL line is up (960 kb/s down | 192 kb/s up)
```

Le premier chiffre est votre débit en download et le second en upload. Au vu de ces chiffres, je peux supposer qu'il y a un soucis sur ma ligne, sachant que le modem speedtouch USB peut monter jusque 8092 kb/s.

Etape 4 : configuration de la connexion.
========================================

Afin d'obtenir du réseau à partir du modem, il faut lancer les commande suivantes, qui permettent à votre linux de connaître sa configuration.

```bash
atmarp -c atm0
ifconfig atm0 <IP_FREE> netmask 255.255.255.0 mtu 1500 up
atmarp -s <GATEWAY_FREE> 8.36 null
route add default gw <GATEWAY_FREE>
```

Aprè ces commandes, vous devriez pouvoir effectuer des ping sur le réseau mondial, tel que vers les serveurs DNS de Google.

```bash
Ping 8.8.8.8
```

Si le ping fonctionne, vous avez de la connexion à Internet. Dans ce cas, n'oubliez pas de configurer vos DNS dans le fichier /etc/resolv.conf afin d'éviter de taper toutes les adresses IP de mémoire.
Si vous souhaitez partager la connexion au sein de votre réseau interne tel une freebox, vous pouvez tenter la configuration d'un réseau temporaire comme je le décris dans mes articles passés.
Liens :

http://nyati-buffaloblog.blogspot.fr/2006/04/ubuntu-speedtouch-howto.html

http://forum.macbidouille.com/lofiversion/index.php/t53852.html
