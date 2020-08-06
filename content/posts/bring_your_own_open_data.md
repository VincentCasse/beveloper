---
title: "Bring your own open data"
date: 2017-05-19T22:16:48+02:00
draft: false
---

Je vis avec une institutrice professeur des écoles. Je n'ai aucun soucis avec cela bien que chaque année, vers le mois d'avril, elle se prend la tête sur le mouvement. Vous ne connaissez pas ? C'est la procédure permettant aux professeurs des écoles de demander une mutation ou d'accéder à une classe fixe. Et comme en début de carrière, il faut attendre quelques années pour obtenir un poste fixe, le mouvement est une étape obligatoire depuis quelques années.

Alors, il faut sélectionner les meilleures écoles, celle dont les chances sont plus élevées que d'autre, mais aussi celle qui sont proche du domicile. Bizarrement, elle n'a pas envie de donner classe dans l'école la plus éloignée du département. Il faut prêt de deux heures pour le traverser d'un point à l'autre ! Sauf que chaque année, c'est la galère : l'éducation nationale propose la liste de tous les postes du département ... triés par ville, par ordre alphabétique. 1034 écoles à placer sur Google Maps pour savoir si le poste peut convenir.

Cette année, j'ai décidé d'innover pour éviter cette galère.

L'an passé, nous avions passé trois soirées consécutives pour trouver les écoles les plus proches de notre lieu de résidence à cause de cette liste triée par ordre alphabétique. En tant que développeur, j'ai une déformation professionnelle : je réfléchi à partir des données que j'observe. Une liste triée par ville. Et non par proximité géographique. On manque quelquechose non ? Je m'étais dit que l'affichage de ces postes sur une carte serait tout de même bien plus pratique. Et pour aider ma compagne, mais aussi tous ces collégues du département, j'ai décidé de tenter l'expérience cette année. Voici comment je m'y suis pris pour le réaliser.

Alors, au premier abord, on se dit qu'en 2017, à force d'entendre parler d'open data dans les communications des administrations et des gouvernements, la liste des postes et leur emplacement serait simple à trouver et qu'il suffirait de les afficher sur une carte. En fait, non, ce serait trop simple ! Alors pour récupérer la liste des postes, il faut regarder comment elle est proposée aux professeurs des écoles. Voici la recette pour créer votre propre open data quand la data n'est pas encore open.

Chaque année, la liste des postes n'est disponible que sur la plateforme permettant de poster ses vœux. Pour chaque poste, l'interface fournit quelques informations uniquement : le nom de l'école, sa ville, son niveau (maternelle ou primaire), le nombre de postes, ainsi que le nombre de postes vacants. Avec cette liste cependant, nous n'avons pas les données nécessaires pour l'afficher sur une carte : ni l'adresse, ni la localisation de l'école...

Au quotidien, on me pose souvent la question "est ce que c'est possible". Quand je suis d'humeur taquine, j'aime répondre qu'en informatique, tout est possible, ce n'est qu'une question de moyens. Alors voilà un challenge intéressant à relever !

Au mois de mars et d'avril, j'ai consacré plusieurs soirées avant de propose en ligne une carte des postes disponibles en Seine Maritime pour la rentrée de 2017. Elle est disponible sur le site Pouka qui vous permet d'afficher les postes disponibles autour de votre localisation. Si vous consultez ce site alors que vous n'habitez pas en Seine Maritime, vous pouvez utiliser les coordonnées suivantes : latitude: 49.5, longitude : 0.76.

Vous rêvez de faire la même chose ? Vous ne savez pas comment créer votre propre Open Data ? Je vais vous décrire, en plusieurs articles de mon blog, comment je m'y suis pris. Et pour ceux qui aiment bien les teasers, le plus difficile n'a pas été l'affichage sur une carte, ni la récupération des données, mais pour le savoir, il faudra lire ces articles en entier ;)

Commençons par l'étape indispensable : la récupération des données

Étape 1 : à la recherche des données perdues
============================================

Le mouvement est une procédure dont la durée est limitée dans le temps. La liste des postes n'est disponible que vers la fin de l'année scolaire et les professeurs des écoles ont environ un mois pour en prendre connaissances et postées leurs souhaits. Vu le temps limité, j'ai décidé de prendre un peu d'avance en recherchant, dés le mois de mars, une solution pour placer les postes sur une carte.

L'avantage des professeurs des écoles, c'est que leur lieu de travail est connu de tous : ils travaillent généralement dans des établissements scolaires et leur liste doit être publique. Effectivement, il existe bien un annuaire comprenant l'ensemble des établissements scolaires, mais ici encore, toujours pas d'open data. Toutes les données seront donc à récupérer par nous même.
La récupération de la liste des écoles

N'ayant pas à notre disposition de données structurées, nous n'avons pas beaucoup de choix : soit l'on remplit notre base de données des écoles manuellement, ce qui va nous prendre bien plus que trois soirées, soit l'on crée un algorithme qui se charge de lire les pages web et de le faire pour nous. En temps que bon développeurs, la paresse étant l'une de nos caractéristiques les plus importante, c'est bien sur la deuxième solution que j'ai sélectionné.

Cette technique, c'est ce que l'on appelle le crawl. C'est exactement ce que font les moteurs de recherche pour comprendre le contenu des pages web et de les référencer. Bref, ce n'est pas nouveau. Mais surtout, il existe une super librairie en python pour crawler et récupérer du contenu au sein de pages web très facilement :

Cette librairie sait parfaitement récupérer le contenu d'une page web, c'est son travail, mais intègre aussi de très nombreuses fonctions permettant de naviguer dans le code source de cette page. Elle utilise par exemple les sélecteurs CSS afin de récupérer des éléments dans la page, ce qui pour toute personne ayant fait du Jquery ou du CSS, rend son usage très simple. Pour tous développeurs web ayant dépassé le stade du développement dans frontpage, cette librairie se prend en main en quelques minutes à peine.

Revenons à notre annuaire. Ce dernier est bien conçu : il permet de rechercher la liste des établissements par niveau scolaire, par localisation... Et pour éviter que l'ensemble de la base ne soit affichée sur une seule page, ce qui la rendrait très lourde à charger, il intègre une pagination. Arf, cela veut dire que nous allons devoir crawler plusieurs pages pour avoir la liste de toutes les écoles du départements.

Et comme un bonheur ne vient jamais seul, la liste des écoles ne comporte toujours pas l'adresse de cette dernière. Il faut cliquer sur le nom de l'école pour récupérer des informations utiles tel que l'adresse, le numéro de téléphone ou bien encore le nombre d'élèves. Encore de nouvelles pages à crawler. GÉ - NI - AL

Encore une fois, la librairie scrapy est là pour nous aider : comme c'est un librairie en python, elle permet de définir dans le code le comportement, et de demander le crawl d'une nouvelle page dont on a trouvé l'url au sein d'une balise <a href=''>

Nous avons donc un algorithme de crawl découpé en deux étapes :

    Le crawl de la page de résultats comprenant la liste des écoles, ainsi que les pages suivantes
    Puis, pour chaque école, le crawl de la page détaillée afin de récupérer les informations nécessaires pour ma cartographie.

Dans scrapy, chaque crawler posséde son code au sein d'un fichier, permettant de séparer des crawls différents. Il faut surtout bien lui définir un nom, permettant de l'appeler ensuite directement depuis la ligne de commande scrapy.

Par exemple, pour analyser le contenu de mon blog, il faut un script avec cette structure minimum :

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

import scrapy
import time

class QuotesSpider(scrapy.Spider):
    name = "blog"

    def start_requests(self):
        urls = [
            'https://beveloper.fr/blog'
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        # Code to parse the blog
```

Pour installer scrapy sur son ordinateur, je vous laisse lire la [documentation|https://scrapy.org/] (mais merci pip). Une fois installé, il est possible de crawler mon blog, et de stocker le résultat souhaité dans le fichier blog.json avec cette commande : 

```bash
scrapy crawl blog -o blog.json
```

Pour l'annuaire des écoles, le crawl se réalise sur deux types de pages différentes. Il faut donc prévoir deux fonctions __parse__ différentes afin d'en extraire les données. La première analyse la liste des écoles et récupère le lien vers la page détaillée, et la seconde analyse les informations détaillées et en extrait les données que nous voulons récupérer sur les écoles. Le tout est envoyé dans le fichier json avec un [générateur|https://wiki.python.org/moin/Generators|en].

Voici le code qui permet d'extraire le lien vers les pages détaillées. Il se trouve dans les balises <a href=''> incluent dans une div avec la classe "annuaire-etablissement-label". Avec le sélecteur CSS, on récupère en une seule ligne l'ensemble des <div> possédant cette classe dans un tableau. Ce qui permet d'itérer très simplement sur toutes les écoles de la page de résultats.

```python
def parse(self, response):
    ecoles = response.css('div.annuaire-etablissement-label')

    for ecole in ecoles:
        link = ecole.css('a::attr(href)').extract_first()
        if link is not None:
            link = response.urljoin(link)
            yield scrapy.Request(link, callback=self.parse_school)
```

Pour chaque école de la liste, on récupére le lien vers la page détaillée au sein de l'attribut href de la balise <a>. Il ne faut pas oublier que le chemin du lien est relatif et il faut donc lui rajouter l'url de l'annuaire avant de lancer notre seconde fonction _parse_

Il faut noter que durant mes tests, j'ai constaté que scrapy réalise son crawl de manière intelligente : il limite le nombre de requêtes simultanées pour avoir le temps de les traiter, et ce qui évite aussi de faire tomber en panne le site que l'on crawl.

L'analyse de la page détaillée sur les écoles est tout aussi simple. Elle nous permet de récupérer l'adresse et quelques informations supplémentaires sur chaque école. A noté que l'utf8 est toujours sympathique à gérer...

```python
def parse_school(self, response):
    
    details = response.css('div.annuaire-etablissement-fiche')

    if details is not None:

        # Clean title
        message = 'École primaire publique '.decode('utf8')
        title = details.css('h2 span::text').extract_first().replace(message, '')
        
        # Address
        addr = details.css('p.annuaire-etablissement-infos-part3::text').extract()
        address = addr0.replace('\n','')
        cityLine = addr1.split(' ')
        postalCode = cityLine0
        city = addr1.replace(postalCode + ' ', '')
        phone = addr3

        result = {
            'maternelle': isMaternelle,
            'primaire'  : isPrimaire, 
            'title'     : title,
            'childrens' : childrens,
            'public'    : isPublic,
            'code'      : code,
            'address'   : address,
            'postalCode': postalCode,
            'city'      : city,
            'phone'     : phone,
        }
        yield result
```

J'ai tronqué un peu le code mais vous pouvez le retrouver en entier sur Github
J'ai aussi oublié de vous expliquer comment crawler facilement en prenant en compte la pagination. En soit c'est simple, il suffit de trouver le lien vers la page suivante, et de relancer un crawl de page de résultat : 

```python
next_page = response.css('.annuaire-pagination li:nth-last-child(2) a')
    
if next_page.css('img') is not None:
    next_page = next_page.css('::attr(href)').extract_first()

    if len(next_page) > 0:
        next_page = response.urljoin(next_page)
        yield scrapy.Request(next_page, callback=self.parse)
```

Il ne reste plus qu'à lancer le crawl avec la commande scrapy et attendre quelques minutes que l'ensemble des pages soient crawlées. Félicitations, vous avez enfin la liste des école en Json dans un fichier. Un bon début pour de l'open data. Passons maintenant aux données suivantes à récupérer.
La récupération de la liste des postes

Sur ce point, je n'ai pas pu prendre d'avance. Il m'a fallu attendre que le mouvement soit commencé avant de regarder comment il serait possible de crawler cette liste. De prime abord, le sujet était plus complexe : cette liste n'est accessible aux professeurs des écoles que sur l'interface du mouvement pour faire ses voeux, et ainsi, nécessite une authentification. Crawler avec authentification, cela peut compliquer sensiblement les choses.

En consultant cette interface, on se rend compte que la page d'authentification est bien plus récente que les pages austères de l'application du mouvement. L'authentification a probablement été mise à jour plus récemment que l'application du mouvement. Mais surtout, les deux applications vivent probablement sur des environnements différents, se partageant l'authentification par SSO.

Pour savoir quelle url il faudrait crawler, j'ai commencé à regarder les url chargées dans mon navigateur et je me suis rendu compte que l'application ne propose pas d'appels AJAX et que la liste est retournée directement dans la page web par le serveur. Très bien, cela sera plus facile à crawler.

En analysant les requêtes, je me suis rendu compte que l'authentification ne serait pas un probléme dans ce cas : le site web du mouvement partage l'identifiant de session non pas par des cookies, comme la plupart des sites web, mais en utilisant un champ SESSION_ID= dans l'url.

Bon, ce n'est pas super au niveau de la confidentialité puisqu'un simple copier coller permet de récupérer une session, mais nous sommes sur un site sécurisé en HTTPS, donc les urls ne passent pas en clair sur le réseau, et la session a une durée de vie limitée, réduisant ce type d'attaque.

L'aspect génial, c'est que cela permet de demander à scrapy de crawler l'url contenant le numéro de session, et permet de récupérer très vite la liste des postes. Enfin, ca simplifie sur les sessions, mais je suis rapidement tombé sur une surprise moins agréable : il existe en réalité une vingtaine de types de postes différents, et il faut sélectionner le type de poste souhaité dans un formulaire pour en récupérer la liste. Il va donc falloir émuler un formulaire dans scrapy. GÉ - NI - AL

Pour cela, il faut utiliser un type de requête scrapy un peut spécial, les FormRequest, qui émulent un formulaire. Rien de bien compliqué dans notre cas puisque pour chaque élement de notre liste, nous émulons le clic sur ce formulaire. Le code suivant permet de le faire rapidement :

```python
posteTypes = response.css('selectname=nature')

for posteType in posteTypes.css('option') :

    code = posteType.css('::attr(value)').extract_first()
    description = posteType.css('::text').extract_first()

    formdata = {
        "action": "SUITE",
        "inature": description.replace(' ', '+'),
        "vac": "*",
        "zon": "*",
        "nat": code,
        "specialite": "*"
    }

    url = 'https://un-bon-crawler-ne-donne-jamais-ses-sources.gouv.fr?SESSION_ID=tutu'

    request = scrapy.FormRequest(url = url, callback=self.parsePoste, formdata=formdata, meta={'code': code})

    yield request
```


Comme le formulaire nous renvoit une page avec un tableau comprenant la liste des postes, il est très simple de le parser et d'en récupérer les informations sur les postes de professeurs des écoles dans le département.

```python
def parsePoste(self, response):

code = response.meta.get('code')

table = response.css('center table tr')

i = 0

postes = 

for tr in table:

    poste = {}

    td = tr.css('td')

    poste'num' = td1.css('::text').extract_first()
    poste'area_type' = td2.css('::text').extract_first()
    poste'name' = td3.css('::text').extract_first()
    poste'type' = td4.css('::text').extract_first()
    poste'specialty' = td5.css('::text').extract_first()
    poste'available' = td6.css('::text').extract_first()
    poste'maybe_available' = td7.css('::text').extract_first()
    poste'bloqued' = td8.css('::text').extract_first()
    poste'quotity' = td9.css('::text').extract_first()
    poste'fulltime' = td10.css('::text').extract_first()

    postes.append(poste)

yield { code: postes }
```

Le code entier est bien sur disponible sur github. Il m'a permis cette année, de récupérer la liste de tous les postes disponible en Seine Maritime à la rentrée, dans un json.

## Point d'étape :

Nous avons désormais la liste de tous les postes disponibles dans un fichier json comprenant les écoles concernées. Pas de quoi les afficher sur une carte puisque nous n'avons pas les adresses des écoles. Mais cette information se trouve dans la fichier des écoles que nous avons crawler sur l'annuaire. C'est assez simple donc : il suffit de fusionner les deux fichiers et nous n'aurons plus qu'à configurer une carte sur un site internet !

Nous verrons si c'est réellement le cas dans le prochain article de blog à propos de Pouka.

A bientôt !