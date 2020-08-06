---
title: "Construction d'un réseau temporaire pour un festival 3"
date: 2011-08-05T22:18:48+02:00
draft: false
---

Pour toutes les personnes n'ayant pas lu les deux articles précédent, il est conseillé de les lire rapidement ici et ici afin de bien comprendre le contexte et l'architecture de ce réseau.

Il ne nous reste plus qu'à configurer la limitation de bande passante attribué au réseau des loges. Rappelez vous : nous aurons besoin lors du festival, d'envoyer un débit issu de la webradio. Ce débit ne doit pas être dégradé, puisqu'il s'agit du lien unique entre le studio et le serveur de rediffusion. Si ce lien est dégradé, c'est l'ensemble de la diffusion qui est touchée.

Après quelques tests, nous sommes arrivés à la conclusion qu'une webradio diffusant à 192kbps utilise un flux réseau d'environ 40Kbits. Notre connexion chez free nous permet d'utiliser jusque 500Kbits en débit montant (upload). C'est dans ce débit que passera le flux de webradio et les requêtes du réseau des loges.

Nous allons donc diviser le réseau en deux : brider le réseau des loges à un maximum 250Kbits et permettre au réseau de la webradio d'utiliser le reste. Les 210kbits de différence permettent d'amortir d'éventuels ralentissements de la ligne ADSL lors du festival. Mais si le réseau descend en dessous de 300kbits, cela deviendra compliqué pour la webradio dans tous les cas.

Revenons à notre serveur. Pour brider le débit du réseau des loges, il faut le faire sur l'interface reliée à la box : ainsi, le bridage sera sur le front montant. De même, si nous devions brider le front descendant, nous le ferions sur l'interface relié au réseau des loges. En effet, le bridage utilisé ne concerne que le front descendant (download) du débit.

Nous allons utiliser un outils qui se nomme "tc" et qui intervient directement sur l'organisation des paquets et en sortie de l'interface réseau. On va donc modifier quelques paramétrages proches du noyau. L'exemple utilisé ici est très simple, on bloque l'ensemble du débit, mais le bridage peut être plus fin ( P2P, retour des ACK de TCP, no limite sur le SSH ...). Pour plus d'informations, allez lire le magazine GNU/Linux Magazine nÂ°127.

Revenons à nos commandes. Sur le serveur, nous créons un nouvel arbre de tri des paquets en sortie de l'interface réseau. Dans cet arbre, chaque branche correspond à un critère (P2P...) défini dans iptable. Si le paquet entrant ne correspond à aucun des critères de branches, il est redirigé vers la branche par défaut, ici la branche 20. Nous n'utiliserons que la branche 20 puisque nous voulons brider TOUT le débit.

```
tc qdisc add dev eth1 root handle 1: htb default 20
```

Nous ajoutons cette règle par défaut (classid 1:20). "rate" représente le débit utilisé en temps normal, "ceil" celui utilisé si du débit est disponible sur le réseau. En gros : l'interface peut aller jusque 200kbits si le réseau est chargé, 250 s'il ne l'est pas.

```
tc class add dev eth1 parent 1:0 classid 1:20 htb rate 200kbit ceil 250kbit prio 1 mtu 1500
```

Petite astuce au passage : si vous devez vider votre arbre en cas de soucis, voici la commande :

```
tc qdisc del dev eth1 root
```

Et voilà, tout est fonctionnel ! Ce n'est pas plus compliqué que cela !  Vous pouvez essayer de regarder si le débit dépasse les 250kbits depuis un poste sur le réseau des loges (http://www.speedtest.net/). Votre débit montant ne dépassera jamais les 250kbits.
Si vous voulez conserver cette configuration, ajoutez ces deux lignes dans le fichier /etc/init.d/filtrage

Nous avons terminé notre petite balade au pays du réseau de festival. Le serveur est actuellement dans les bureaux, en test, en attendant que le festival débute. Nous allons maintenant pouvoir s'occuper du reste de l'infrastructure du festival : webradio et cÃ¢blage du site en vue ! Et si je suis en forme, peut être quelques articles qui explique leurs fonctionnement !

A bientôt !