---
title: "Construction d'un réseau temporaire pour un festival 2"
date: 2011-08-03T22:18:48+02:00
---

Pour tous ceux qui n'ont pas lu le billet précédent, je vous conseille de le lire avant d'attaquer celui-ci : il décrit en bonne partie la conception du réseau, ses contraintes et ses objectifs.
Aujourd'hui, nous allons voir comment permettre le partage de la connexion à internet pour le réseau des loges et comment installer un proxy transparent pour contrôler un peu ce qui passe sur notre réseau et rester dans la légalité (on devient le temps d'un week end, fournisseur d'accè à des personnes que l'on connaît peu).

Partage de la connexion :
=========================

Les boxs ADSL sont des outils merveilleux : elles ont permis de démocratiser des usages qui ne seraient pas accessibles avec des routeurs habituels. Cependant, elles restent assez limitées et nous sommes obligés de passer par elle pour avoir Internet.

Ainsi, nous ne pouvons pas rediriger le trafic vers un autre réseau que celui de la box : impossible de faire du routage avec la freebox.Il nous reste la technologie NAT. Pour ceux qui ne connaissent pas, elle permet de faire passer tout les flux du réseau des loges avec une seule adresse IP : celle de notre serveur. Pas besoin de déclarer à la box qu'il existe un sous réseau, on transfère tout au serveur qui se charge de répartir les flux.

Dans linux, on peut éditer beaucoup de paramètres de la pile réseau. C'est trés précis, mais aussi assez périlleux : une erreur et votre réseau ne fonctionne plus. On va d'abord autoriser la pile réseau à transférer des flux entre les différentes interfaces du serveur. Editez le fichier /etc/sysctl.conf, et décommentez les lignes :

```bash
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Pour gérer les flux entrants et sortants, le noyau linux propose un outil : **iptables**. Il permet de définir un ensemble de régles de gestion du réseau. Pour que ces règles s'appliquent à chaque démarrage du serveur, il faut créer un fichier les contenant, et l'exécuter au démarrage. Le fichier doit se trouver dans /etc/init.d/ : 

```bash
#!/bin/sh
#configuration de iptables
# on vide les chaines
iptables -F

#On interdit tous les accés en entrée
iptables -P INPUT DROP

#Tout ce qui est en sortie, ou routé est accepté
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# Pas de filtrage depuis le réseau externe et la pile interne et le ping
iptables -A INPUT -i eth1 -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT

#Accepter les paquets de flus déjà actifs
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

#Ouvrir des ports (web, mail, ssh, dhcp, dns ..)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p udp --dport 67 -j ACCEPT
iptables -A INPUT -p udp --dport 68 -j ACCEPT
iptables -A INPUT -p udp --dport 69 -j ACCEPT
iptables -A INPUT -p udp --dport domain -j ACCEPT
iptables -A INPUT -p tcp --dport domain -j ACCEPT
iptables -A INPUT -p tcp --dport ssh -j ACCEPT
iptables -A INPUT -p tcp --dport smtp -j ACCEPT
iptables -A INPUT -p tcp --dport smtps -j ACCEPT
iptables -A INPUT -p tcp --dport pop3 -j ACCEPT
iptables -A INPUT -p tcp --dport pop3s -j ACCEPT
iptables -A INPUT -p tcp --dport imap -j ACCEPT
iptables -A INPUT -p tcp --dport imaps -j ACCEPT
iptables -A INPUT -p tcp --dport https -j ACCEPT

#Redirection du port 80 vers le proxy (port 3128)
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 3128

# Faire du NAT
iptables -t nat -A POSTROUTING -s 10.1.0.0/16 -o eth1 -j MASQUERADE
```

Une fois le fichier enregistré, rendez le executable et ajoutez le au démarrage :

```bash
chmod +x /etc/init.d/filtrage
update-rc.d filtrage defaults 99
```

Dans ce fichier de configuration iptables, nous avons ajouté une redirection : on redirige le flux du port 80 vers le port 3128. C'est sur ce port que nous allons installer le proxy. Cela permet de le rendre transparent : aucune configuration nécessaire sur les appareils des utilisateurs : n'importe qui sur le réseau, passera par le proxy.

L'installation du proxy est relativement simple tant que l'on ne filtre pas les sites internet :

```bash
apt-get install squid-common
```

Un conseil : sauvegardez le fichier de configuration par défaut et créez un nouveau fichier :

```bash
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
nano /etc/squid/squid.conf
```

```bash
Voici ma configuration par défaut :
# Déclaration du port du proxy
http_port 3128 transparent

#Définition des ACLs pour autoriser ou refuser l'origine d'un flux
acl all src all
acl localhost src 127.0.0.1
acl lan src 10.1.0.0/16

# Refus ou autorisation en fonction des acl
http_access allow localhost
http_access allow lan
http_access deny all

#Enregistrement des logs
access_log /var/log/squid/access.log squid
```

Redémarrez (commande reboot) et, si tout va bien, vous aurez accè à internet depuis le réseau des loges.
Pour vérifier que vous êtes bien passés par le proxy, ouvrez le fichier /var/log/squid/access.log, toutes vos connexions doivent y être inscrites.

Installation du filtrage des sites :
====================================

L'avantage de squid, c'est surtout le module squidGuard que l'on peut ajouter : il permet de bloquer certains sites web en fonction d'une liste noire de sites. La liste la plus connue (pour la France) est maintenant par l'université de Toulouse. Voici comme cela s'installe :

```bash
apt-get install squidguard
```

Nous allons ensuite charger la liste de sites, qui est classé par type de site. Nous ne bloquerons ainsi que les sites illégaux selon la législation FranÃ§aise et les sites qui permettent de contourner le proxy.

```bash
cd /var/lib/squidguard/db
wget ftp://ftp.univ-tlse1.fr/pub/reseau/cache/squidguard_contrib/blacklists.tar.gz
tar zxvf blacklists.tar.gz
mv blacklists/* .
chown -Rf proxy:proxy .
```

Il fait ensuite définir quels sont les sites interdits. Cela se passe dans le fichier /etc/squid/squidGuard.conf :

```
dbhome /var/lib/squidguard/db
logdir /var/log/squid

dest agressif {
        domainlist      agressif/domains
        urllist         agressif/urls
        expressionlist  agressif/expressions
        log             agressif
}
dest dangerous_material {
        domainlist      dangerous_material/domains
        urllist         dangerous_material/urls
        log             dangerous_material
}
dest gambling {
        domainlist      gambling/domains
        urllist         gambling/urls
        log             gambling
}
dest malware {
        domainlist      malware/domains
        urllist         malware/urls
        expressionlist  malware/expressions
        log             malware
}
dest phishing {
        domainlist      phishing/domains
        urllist         phishing/urls
        log             phishing
}
dest reaffected {
        domainlist      reaffected/domains
        urllist         reaffected/urls
        log             reaffected
}
dest redirector {
        domainlist      redirector/domains
        urllist         redirector/urls
        expressionlist  redirector/expressions
}
dest strong_redirector {
        domainlist      strong_redirector/domains
        urllist         strong_redirector/urls
        expressionlist strong_redirector/expressions
        log             strong_redirector
}
dest sect {
        domainlist      sect/domains
        urllist         sect/urls
        log             sect
}

acl {
        default {
                pass     !malware !phishing !reaffected !redirector !sect !agressif !dangerous_material !gambling !strong_redirector all
                redirect 302:http://www.lamareathon.com/interdit.html
        }
}
```

Tous les "dest" définissent les catégories de sites. Vous pouvez retrouver toutes les catégories disponibles en affichant la liste des dossiers dans /var/lib/squidguard/db/. Une catégorie = un dossier.
"acl" définit la régle de validation ou non du site. Tout ce qui est précédé de "!" ne passe pas. Le "all" a la fin de la ligne indique que tous les autres sites sont valides. La ligne "redirect" indique la page qui est affichée à l'utilisateur lorsqu'un site web est interdit.

Pour tester si votre squidGuard fonctionne, je vous conseille de lancer la commande

```bash
/usr/bin/squidguard -C /etc/squid/squidGuard.conf -b
```

qui vous affiche l'état de chargement des listes de fichiers. Si jamais le chargement se bloque, regardez le fichier de log /var/log/squid/squidGuard.log, il y a souvent l'explication du blocage.
Enfin, si tout fonctionne, il ne reste plus qu'à indiquer à Squid qu'il doit transmettre les informations à squidGuard. On se retrouve à ajouter à la fin du fichier /etc/squid/squid.conf les lignes suivantes :

```bash
# Squidguard
redirect_program /usr/bin/squidGuard -c /etc/squid/squidGuard.conf  
redirect_children 10
```

Relancez votre squid, patientez quelques minutes que squidGuard charge ces listes en mémoire (vous n'aurez pas accè aux sites durant le chargement, depuis le réseau des loges), puis testez avec un site qui est dans la liste des sites à bloquer.

Pour que notre réseau soit totalement fonctionnel, il ne restera plus que la configuration du bridage de débità faire. Mais pas ce soir !

A bientôt,

Lien utile lors de ma configuration : http://coagul.org/drupal/node/139/