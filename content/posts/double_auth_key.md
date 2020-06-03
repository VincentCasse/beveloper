---
title: "La double authentification par clé : comment ca marche ?"
date: 2016-05-28T22:18:48+02:00
---

En mars dernier, j'ai réalisé deux talks au Breizhcamp : une conférence pour les développeurs qui se déroule tous les ans à Rennes. Durant la keynote du jeudi, nous avons tous récupéré une clé U2F permettant de sécuriser nos comptes. J'ai voulu tester mais ca n'a pas été si simple. Petit retour d'expérience.

Avant de débuter mon retour d'expérience, pourquoi est ce utilise de s'authentifier par clé ?

# À quoi ca sert ?

L'authentification par login / mot de passe n'est pas fiable. Le login est rarement secret et il suffit donc de connaître le mot de passe pour accéder au compte souhaité. Bien sur, ce n'est pas si simple de connaître le mot de passe, mais entre les utilisateurs qui utilisent des mots de passe faibles et les bases de données qui fuitent .. c'est devenu chose courante. D'ailleurs petite astuce : évitez d'utiliser le même mot de passe sur deux sites différents...

Depuis quelques années, la double authentification permet de compléter le duo login / mot de passe. Souvent, il s'agit d'une application que l'on configure sur son téléphone. Pour se connecter, il faut donc le login et mot de passe, mais aussi utiliser le code fourni par le téléphone, et qui a une durée de vie limitée. L'intérêt de la double authentification est de se baser sur un périphérique physique pour compléter la connexion. C'est dur de voler un téléphone par Internet...

Mais le téléphone n'est pas le seul type de double authentification. Il existe un type de clé usb, respectant un protocole nommé U2F qui permet de valider une authentification. L'avantage par rapport au smartphone : pas besoin de le sortir à chaque connexion à un site, une fois que la clé est connectée à l'ordinateur, tous les sites sécurisés avec cette clé peuvent établir une connexion. C'est ce type de clé qui était offert au Breizhcamp

{{< tweet 712923623003144193 >}}


Vous n'étiez pas au Breizhcamp ? Ne vous inquiétez pas, on trouve des clés U2F pour quelques euros sur Internet comme sur amazon .

# Bon alors comment on l'utilise ?

C'est là que le bât blesse. Il faut remplir plusieurs conditions : utiliser un navigateur compatible et un site qui intégre ce fonctionnement. Si vous développez des sites, je reviendrais dans un autre article sur le fonctionnement de ce protocole.

Actuellement, seul la version stable de Chrome (40+) intégre le protocole U2F par défaut. Ce qui est pas si étonnant : Google fait parti des organisations à l'initiative de U2F. Mozilla Firefox ne l'intégre pas par défaut mais il est possible d'installer l'extension U2F support. L'implémentation native a longtemps été mise en standby mais ca bouge depuis le début de l'année et si vous utilisez la version developer de Firefox, vous pouvez déjà activer le support natif du protocole comme l'a indiqué @rlbarnes.

{{< tweet 728298931935916034 >}}

Maintenant que l'on a le navigateur, il faut trouver les sites compatibles. Sans grand étonnement, l'authentification de Google gère U2F. Mais vous pouvez aussi jouer avec Github.

Pour Google, il faut se rendre dans les paramétres de son compte, sur la page "Ajouter une clé de sécurité", puis demander l'enregistrement de la clé. À ce moment, il faut connecter votre clé à l'ordinateur et Google la détectera. Et c'est tout. La clé est enregistrée par Google, et sera l'une des solutions de double authentification disponible au prochain login.
Pour plus d'informations, vous pouvez consulter la documentation officielle de Google

Pour Github, le processus est similaire : il faut déjà avoir la double authentification d'activée et il faut se rendre dans les paramètres de compte, partie "Sécurité" pour enregistrer une nouvelle clé. De la même manière, au prochain login, vous aurez le choix entre la double authentification par sms ou par votre clé.
C'est simple, pourquoi tout le monde ne l'utilise pas ?

Ce n'est pas dur à utiliser, mais le nombre de conditions à remplir est important : il faut le bon navigateur, que le site le gére et avoir une clé U2F à sa disposition. En gros, il faut que son usage se développe pour que les sites s'y mettent. Mais pour que l'usage se développe, le nombre de sites qui le gére doit augmenter. Un vrai problème d'oeuf et de poule. Bref, il ne faut pas hésiter à en parler, à le tester, afin de pousser les sites à l'utiliser. Achetez une clé, configurez là, utilisez là, demandez aux sites d'intégrer ce processus de login. L'initiative du Breizhcamp d'offrir des clé U2F va dans ce sens. Sans le Breizhcamp, pas cet article ! Et vous, comment pouvez vous aider ?

Le prochaine étape : comment je le code pour mon site ?

A bientôt,