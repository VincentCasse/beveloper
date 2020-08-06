---
title: "InvalidArgumentException The target directory 'www' does not exist"
date: 2013-05-15T22:18:48+02:00
draft: false
---

Avec Symfony2 sur un serveur mutualisé d'OVH (offre pro), il est possible que vous obteniez l'erreur suivante : 

```bash
php5.3 app/console assets:install www                               
[InvalidArgumentException]                 
The target directory "www" does not exist. 
```                                             

Pour utiliser cette commande, vous devez utiliser le chemin absolu du répertoire :

```bash
pwd
/homez.***/e***
```

```bash
php5.3 app/console assets:install /homez.***/et***/www --symlink
Installing assets using the symlink option
Installing assets for Symfony\Bundle\FrameworkBundle into /homez.713/etretata/www/bundles/framework
Installing assets for Sensio\Bundle\DistributionBundle into /homez.713/etretata/www/bundles/sensiodistribution
```