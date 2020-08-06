---
title: "Changer son mot de passe root en cas de secours"
date: 2011-08-03T22:16:48+02:00
draft: false
---

Bonsoir à tous,

Pour ceux qui ne connaissent pas linux, le compte root est le compte utilisateur le plus important de linux : c'est le super administrateur. Il a tout les pouvoirs, y compris d'écrire dans les périphériques et de les abimer ! Il faut donc faire attention avec ce compte et y mettre un bon mot de passe.

Cependant, il arrive de perdre ce mot de passe. La technique est de le changer avant le démarrage du système. 

Cependant, il vous faudra un accés physique sur le système.

Vous aurez raison de me dire que pouvoir changer le mot de passe au démarrage est une faille, mais il est parfois plus simple de ce laisser cette solution de secours que de perdre tout accè sur une machine ou un serveur. Pour empêcher l'exploitation de cette faille, il faut donc empecher l'accè physique au poste.
Revenons à la méthode de changement du mot de passe. Lors du démarrage, vous avez (sous Ubuntu et Debian, par défaut), GRUB qui vous permet de choisir votre linux. Faîtes "e" pour éditer le script. 

A la ligne se terminant par :

```bash
ro quiet
```

Le changer en : 

```bash
rw quiet init=/bin/sh
```

**Attention : Le clavier est en qwerty.**

Lancez le système. Vous arrivez rapidement à une console. Faîtes

```bash
passwd
```


Et indiquez votre nouveau mot de passe. Tapez

```bash
reboot
```


Et votre système rédémarre. Vous aurez maintenant accè au compte root avec votre nouveau mot de passe. N'oubliez pas que lorsque vous tapez votre mot de passe, le clavier est en qwerty, ce qui ne sera pas le cas au redémarrage.
C'était la petite astuce du soir ! Trè connue, mais toujours pratique !

Bonne soirée !