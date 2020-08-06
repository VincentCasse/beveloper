---
title: "Project Dish : Gestion de webradio"
date: 2011-08-01 12:30:00+02:00
draft: false
---

Il y a de cela quelque mois, l'association Art en Sort a décidé de mettre en place une webradio. Sortie en même temps que son nouveau site Internet, cette webradio doit permettre de diffuser des émissions réalisés pour le plaisir par les adhérents, mais aussi lors d'animations proposées par l'association aux structures de la région (Lycée, centre de loisirs ...)

J'ai donc été chargé de mettre en place cette webradio. Du coté serveur, rien de très spécifique : un serveur dédié avec un icecast dessus et un peu de monitoring afin de vérifier que les ressources ne sont pas limites. Les premiers résultats sont très positifs, un serveur dédié doit pouvoir tenir la charge de plusieurs centaines d'utilisateurs simultanés.

Du coté client, j'ai eu beaucoup plus de soucis : ma recherche était assez restrictive : il me faut un logiciel qui puisse être utilisé par tout le monde. Que ce soit, sous windows ou sous linux, il n'y a pas d'importance. Même le payant ne me posait pas trop de problèmes ...  
Mais il n'y a pas beaucoup de logiciels client simple : soit ils permettent seulement de gérer la playlist, soit ils permettent de gérer uniquement le live, soit ils ne sont pas stable.

J'ai alors découvert un outils merveilleux mais complexe : liquidsoap. C'est un langage de script qui permet de mêler et de dissocier des flux entre eux : que ce soit des fichiers situés sur le poste ou bien des flux de la carte son, on peut faire beaucoup de choses avec. Cependant, les interfaces graphiques existantes ne sont pas finies et les outils de gestion de webradio basées sur liquidsoap manquent cruellement de gestion du live.

Pour les besoins de l'association, j'ai donc codé une interface graphique en PHP (seul langage que je connais assez bien pour faire rapidement ce projet) avec Symfony pour ne pas s'occuper de la couche MVC. C'est moche, c'est pas du tout perfectionné mais cela permet de gérer un peu la webradio. Ce n'est qu'une interface pour liquidsoap et il doit donc être utilisé uniquement sur le poste oÃ¹ il est installé puisqu'il gère le micro local. L'accés àdistance doit être géré de manière très délicate.

Voici quelques screenshots et le lien pour télécharger Dish. Le projet est hébergé sur Github et un fichier d'installation a été étudié pour automatiser l'installation sous Ubuntu 10.10.

[![<screen>](http://project-dish.liberetongeek.com/Dish1.png)](http://project-dish.liberetongeek.com/Dish1.png) [![<screen>](http://project-dish.liberetongeek.com/Dish2.png)](http://project-dish.liberetongeek.com/Dish2.png) [![<screen>](http://project-dish.liberetongeek.com/Dish3.png)](http://project-dish.liberetongeek.com/Dish3.png)

[Download](project-dish.liberetongeek.com/dish.tar.gz)

Ce logiciel n'est pas terminé, et j'ai beaucoup de modifications àfaire si j'en prenais le temps, voici une liste non exhaustive. N'hésitez pas, si vous cherchez àl'utiliser, àme poser des questions dans les commentaires : je n'ai jamais installé dish en dehors de ma configuration initiale.

**TodoList Dish :**

\- Changer le moteur liquidsoap par Mplayer (liquidsoap rame beaucoup trop, lâinterfaÃ§age de Mplayer avec du C peut permettre de meilleures performances et de créer des fonctions précises)  
\- Permettre une meilleure gestion de la bibliothèque : différents tri, notation ...  
\- Revoir le système de musique aléatoire pour éviter qu'elles se rediffusent trop souvent  
\- Installer un système d'authentification pour enregistrement des configurations par émissions  
\- Permettre la programmation de fichier enregistrés pour diffuser les émissions àdes heures précises  
\- Permettre la diffusion de spots de manière régulière.