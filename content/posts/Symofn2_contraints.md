---
title: "Symfony2, contraintes et Entity non valides"
date: 2011-08-05T22:18:48+02:00
---

Petite astuce du soir pour le développement sous Symfony 2 : 

Vous avez un problème avec vos entités lors du rechargement de votre cache ?

    is not a valid entity or mapped super class

Vous n'arrivez pas à appliquer les contraintes de vos entity en annotations, lors de l'envoi de votre formulaire ?

    /**
     * @var UploadedFile $file
     * @Asset\File( maxSize = "1024k",
     *     mimeTypes = {"image/jpeg", "image/png", "image/gif"},
     *     mimeTypesMessage = "Ce fichier doit être une image")
     */

(Vous êtes sous Fedora ?)

Regarder dans votre profiler si eAccelerator est "enabled". Si c'est le cas ... c'est probablement la cause de pas mal de vos soucis !

Pour Fedora, il m'a suffit de faire

    yum remove php-eaccelerator

Et j'ai retrouvé un symfony stable, un peu moins rapide, mais sans bugs bizarres pour le développement.