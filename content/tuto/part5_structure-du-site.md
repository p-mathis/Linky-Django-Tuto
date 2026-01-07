+++
date = '2025-12-17T10:42:24+01:00'
draft = false
title = 'Partie 5 : Structure Du Site'
description = "Notion de mode MTV. Écriture du fichier views.py avec les différentes fonctions, des templates (gabarits) qui vont constituer le site pour visualiser les données Linky"
+++
{{< line >}}
## Principe de fonctionnement de Django
### Pour en savoir plus sur Django
- [djangoGirls](https://tutorial.djangogirls.org/fr/)
- [mozilla](https://developer.mozilla.org/fr/docs/Learn_web_development/Extensions/Server-side/Django)
### Mode url - view - template
Django fonctionne selon le mode [MTV (model/view/template)](https://www.tresfacile.net/architecture-mtv-de-django/#1) :
- Une {{< focus >}}URL{{< /focus >}} appelle une {{< focus >}}view{{< /focus >}} ; la {{< focus >}}view{{< /focus >}} traite la logique et renvoie à l'affichage dans le navigateur un {{< focus >}}template{{< /focus >}}  
Pour chaque page que nous souhaitons afficher il faut donc : 
- créer une vue dans {{< focus >}}~/djangoTIC/ticServer/ticapp/views.py{{< /focus >}} sous forme d'une [fonction](https://courspython.com/fonctions.html)
- créer une url dans {{< focus >}}~/djangoTIC/ticServer/ticapp/urls.py{{< /focus >}} qui sera spécifique pour la vue
- créer un gabarit (template) dans {{< focus >}}~/dajngoTIC/ticServer/ticapp/template/{{< /focus >}}, spécifique pour la vue et l'url considérées  
Dans ce tutoriel, nous allons créer les pages :
- accueil : affichage des puissances apparentes instantanées
- jour : affichage de la consommation depuis minuit pour le jour en cours
- 24 heures : affichage de la consommation des 24 dernières heures
- hier : affichage de la consommation de la veille
- dernière heure : affichage de la consommation de l'heure écoulée
{{< line >}}
## Création de la page d'accueil
La page d'accueil {{< focus >}}index.html{{< /focus >}} va afficher :
- l'heure actuelle fournie par la Raspberry
- la puissance apparente totale soutirée atuellement
- les puissances apparentes soutirées actuellement pour chaque phase
- le type de tarif : heures creuses / heures pleines
- la date donnée par le Linky (sous forme yymmjjhhmmss)
### Fonction *index*
- Le code de la fonction index est le suivant : 
{{< codefile file="ticServer/ticapp/views.py" lang="python" >}}
def index(request):
    now = timezone.now()
    data = Data.objects.last()
    total = format(data.sinsts, ',').replace(',', ' ')
    phase1 = format(data.sinsts1, ',').replace(',', ' ')
    phase2 = format(data.sinsts2, ',').replace(',', ' ')
    phase3 = format(data.sinsts3, ',').replace(',', ' ')
    date = data.date
    creuxplein = data.ltarf
    values = {
        'total': total, 
        'phase1': phase1,
        'phase2': phase2,
        'phase3': phase3,
        'date': date,
        'creuxplein': creuxplein,
    }

    context = {
        'values': values,
        'now': now,
    }

    return render(request, 'index.html', context)
{{< /codefile >}}
### Remarques sur le code 
- Pour simplifier la gestion des dates, on utilise le module {{< focus >}}timezone{{< /focus >}} de {{< focus >}}django.utils{{< /focus >}} ; il faut importer ce module :  
{{< cmd >}}from django.utils import timezone{{< /cmd >}}
- On cherche le dernier enregistrement dans la table de la base de données : {{< focus >}}data = Data.objects.last(){{< /focus >}} ; il faut importer notre modèle {{< focus >}}Data{{< /focus >}} :  
{{< cmd >}}from .models import Data{{< /cmd >}}
- On formate les différentes valeurs de manière à remplacer les séparateurs de milliers {{< focus >}},{{< /focus >}} (format anglo-saxon) par un espace
- On retourne un dictionnaire {{< focus >}}values{{< /focus >}} et la date/heure actuelle (le contexte) vers le gabarit {{< focus >}}index.html{{< /focus >}}
### Modifier le fichier *views.py*
- Ouvrir le fichier : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/views.py {{< /cmd >}}
- Copier le code suivant :
{{< codefile file="ticServer/ticapp/views.py" lang="python" >}}
from django.shortcuts import render
from django.utils import timezone           # new
from .models import Data                    # new

def index(request):                         # new
    now = timezone.now()
    data = Data.objects.last()
    total = format(data.sinsts, ',').replace(',', ' ')
    phase1 = format(data.sinsts1, ',').replace(',', ' ')
    phase2 = format(data.sinsts2, ',').replace(',', ' ')
    phase3 = format(data.sinsts3, ',').replace(',', ' ')
    date = data.date
    creuxplein = data.ltarf
    values = {
        'total': total, 
        'phase1': phase1,
        'phase2': phase2,
        'phase3': phase3,
        'date': date,
        'creuxplein': creuxplein,
    }

    context = {
        'values': values,
        'now': now,
    }

    return render(request, 'index.html', context)
{{< /codefile >}}
- *(Enregistrer et quitter)*
### Créer l'url de la page
#### Créer le fichier *ticapp/urls.py*
- Dans {{< focus >}}ticapp/{{< /focus >}}, il faut créer (il n'existe pas par défaut) un fichier {{< focus >}}urls.py{{< /focus >}} qui va contenir les url des différentes pages du site : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/urls.py{{< /cmd >}}
- On crée l'url de la page *accueil* :
{{< codefile file="ticServer/ticapp/urls.py" lang="python" >}}
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
{{< /codefile >}}
- *(Enregistrer et quitter)*
#### Commentaire sur le code
- On a appellé la fonction {{< focus >}}index{{< /focus >}} de {{< focus >}}views.py{{< /focus >}}
- Le chemin est {{< focus >}}''{{< /focus >}} : ainsi lorsqu'on tape l'adresse du site sans ajouter un nom de page, on tombe directement sur cette vue
- Le nom {{< focus >}}name{{< /focus >}} a peu d'importance dans ce tutoriel
#### Modifier le fichier *ticServer/urls.py*
- Il faut indiquer au fichier *ticServer/urls.py* qu'il doit prendre en compte les url du fichier {{< focus >}}ticapp/urls.py{{< /focus >}}
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticServer/urls.py{{< /cmd >}}
- On ajoute le chemin vers les urls de {{< focus >}}ticapp/urls.py{{< /focus >}} : {{< cmd >}}path('', include('ticapp.urls')){{< /cmd >}}
- Et comme on utilise le module {{< focus >}}include{{< /focus >}} on importe celui-ci
- Le code du fichier {{< focus >}}ticServer/urls.py{{< /focus >}} devient :
{{< codefile file="ticServer/ticServer/urls.py" lang="python" >}}
from django.contrib import admin
from django.urls import path, include   # include : new

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('ticapp.urls')),   # new    
]
{{< /codefile >}}
- *Enregistrer et quitter*
### Structure des gabarits *(templates)*
Les pages html ont une structure répétitive avec : 
- un [head](https://developer.mozilla.org/fr/docs/Web/HTML/Reference/Elements/head)
- un menu
- le contenu spécifique de la page
- éventuellement un [pied de page (ou footer)](https://developer.mozilla.org/fr/docs/Web/HTML/Reference/Elements/footer) 
Les éléments communs à chaque page (*head*, menu, pied de page) vont être codés dans un (des) fichier(s) spécifique(s) de façon à ne pas être codés dans toutes les pages.
- Créer le dossier *templates* : {{< cmd >}}mkdir ~/djangoTIC/ticServer/ticapp/templates{{< /cmd >}}

### Fichier *_base.html*
- Créer en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/_base.html{{< /cmd >}}
- Copier le code :
{{< codefile file="ticapp/templates/_base.html" lang="html" >}}
{% load static %}

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TIC Server</title>
    <link rel="stylesheet" href="{% static 'src/output.css' %}">
</head>
<body class="bg-green-50">
    <div class="container mx-auto mt-4">
        {% block content %}
        {% endblock content %}
    </div>
    <script src="{% static 'flowbite/dist/flowbite.min.js' %}"></script>    
</body>
</html>
{{< /codefile >}}
- *Enregistrer et quitter*

### Commentaires sur le code
- {{< focus >}}{% load static %}{{< /focus >}} indique qu'il faut charger les fichiers statiques (notamment le fichier *css*)
- dans le *head* on appelle le fichier *css* par la [balise link](https://developer.mozilla.org/fr/docs/Web/HTML/Reference/Elements/link) en respectant son chemin
- {{< focus >}}{% block content %}{{< /focus >}} va insérer la valeur du bloc *content* qui correspond au contenu des différentes pages
- Juste avant de fermer la [balise body](https://developer.mozilla.org/fr/docs/Web/HTML/Reference/Elements/body), on insère le [script](https://developer.mozilla.org/fr/docs/Web/HTML/Reference/Elements/script) *flowbite* qui sera utilisé, notamment dans le *menu*
### Fichier *menu.html*
- Créer en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/menu.html{{< /cmd >}}
- Copier le code : 
{{< codefile file="ticapp/templates/menu.html" lang="html" >}}
    {% load static %}
    <nav class="bg-green-50 border-gray-200 px-2 sm:px-4 py-2.5 rounded dark:bg-gray-800">
        <div class="container flex flex-wrap items-center justify-between mx-auto">
        <a href="{{ .Site.Params.homepage }}/" class="flex items-center">
            <img src="{% static 'images/linkylogo.png' %}" class="h-6 mr-3 sm:h-9" alt="Site Logo" />
            <span class="self-center text-xl font-semibold whitespace-nowrap dark:text-white">TIC Serveur</span>
        </a>
        <button data-collapse-toggle="mobile-menu" type="button" class="inline-flex items-center p-2 ml-3 text-sm text-gray-500 rounded-lg md:hidden hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-gray-200 dark:text-gray-400 dark:hover:bg-gray-700 dark:focus:ring-gray-600" aria-controls="mobile-menu" aria-expanded="false">
            <span class="sr-only">Open main menu</span>
            <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M3 5a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM3 10a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM3 15a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z" clip-rule="evenodd"></path></svg>
            <svg class="hidden w-6 h-6" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
        </button>
        <div class="hidden w-full md:block md:w-auto" id="mobile-menu">
            <ul class="flex flex-col mt-4 md:flex-row md:space-x-8 md:mt-0 md:text-sm md:font-medium">
            <li>
                <a href="{% url 'index' %}" class="block py-2 pl-3 pr-4 text-white bg-zinc-400 rounded md:bg-transparent md:text-green-700 md:p-0 dark:text-white" aria-current="page">Accueil</a>
            </li>
            <li>
                <a href="{% url 'jour' %}" class="block py-2 pl-3 pr-4 text-gray-700 border-b border-gray-100 hover:bg-gray-50 md:hover:bg-transparent md:border-0 md:hover:text-green-700 md:p-0 dark:text-gray-400 md:dark:hover:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">Jour</a>
            </li>
            <li>
                <a href="{% url 'lasthour' %}" class="block py-2 pl-3 pr-4 text-gray-700 border-b border-gray-100 hover:bg-gray-50 md:hover:bg-transparent md:border-0 md:hover:text-green-700 md:p-0 dark:text-gray-400 md:dark:hover:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">Dernière Heure</a>
            </li>
            <li>
                <a href="{% url 'last24hours' %}" class="block py-2 pl-3 pr-4 text-gray-700 border-b border-gray-100 hover:bg-gray-50 md:hover:bg-transparent md:border-0 md:hover:text-green-700 md:p-0 dark:text-gray-400 md:dark:hover:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent dark:border-gray-700">24 heures</a>
            </li>
            <li>
                <a href="{% url 'hier' %}" class="block py-2 pl-3 pr-4 text-gray-700 hover:bg-gray-50 md:hover:bg-transparent md:border-0 md:hover:text-green-700 md:p-0 dark:text-gray-400 md:dark:hover:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent">Hier</a>
            </li>
            </ul>
        </div>
        </div>
    </nav>
{{< /codefile >}}
- *Enregistrer et quitter*
- Dans un premier temps, on ne lance pas ce menu dans le fichier {{< focus >}}_base.html{{< /focus >}} ; effectivement, cela engendrerait un bogue, les différentes url du menu n'étant pas encore créées.
- Le code du menu est standard, s'adaptant aux écrans de petite taille (*responsive*) ; il aurait pu être amélioré en codant l'affichage de la liste des entrées. Comme il y a relativement peu de pages, le choix paresseux a été de tout écrire en dur.
#### Image logo
Pour agrémenter le menu, une image a été insérée : {{< focus >}}linkylogo.png{{< /focus >}}. Cette image doit être copiée dans le dossier {{< focus >}}~/djangoTIC/ticServer/ticapp/static/images{{< /focus >}}.  
- Utiliser la commande [wget](https://doc.ubuntu-fr.org/wget)
- Télécharger l'image *linkylogo.png* : {{< cmd >}}wget -O ~/djangoTIC/ticServer/ticapp/static/images/linkylogo.png https://linky-tuto.netlify.app/images/linkylogo.png{{< /cmd >}}

### Fichier *index.html*
- Créer en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/index.html{{< /cmd >}}
- Copier le code : 
{{< codefile file="ticapp/templates/index.html" lang="html" >}}
{% extends "_base.html" %}
{% load l10n %}
{% load tz %}

{% block content %}
<div class="max-w-3xl mx-auto text-center mt-12">
    <h1 class="text-xl md:text-3xl lg:text-4xl font-bold text-gray-700 leading-tight mb-2 border-b-2 border-gray-400 pb-2">
      Puissance Instantanée (en Watt)
    </h1>
</div>

{% timezone "Europe/Paris" %}

<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-semibold text-gray-500">{{ now | date:" D j F Y - H:i:s "}}</p>
    <p class="text-gray-400 pl-4"></p>
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="text-gray-400 ">Date du TIC : {{ values.date }}</p>
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="text-gray-400 ">{{ values.creuxplein }}</p>
</div>
{% endtimezone %}

<div class="flex items-center justify-center pt-2 md:pt-4">
    <table class="border-separate border-spacing-x-8 md:border-spacing-x-16 border-spacing-y-4 md:border-spacing-y-8 border border-gray-200">
        <tbody>
            <tr class="">
                <td class="font-semibold ">Total 3 phases</td>         
                <td class="font-semibold text-right ">{{  values.total }}</td>
            </tr>
            <tr>
                <td class="">Phase 1</td>
                <td class="text-right">{{  values.phase1 }}</td>
            </tr>
            <tr>
                <td class="">Phase 2</td>  
                <td class="text-right">{{  values.phase2 }}</td>
            </tr>
            <tr>
                <td class="">Phase 3</td>
                <td class="text-right">{{  values.phase3 }}</td>
            </tr>
        </tbody>
    </table>
</div>

<script>
    setTimeout(function(){
       window.location.reload(1);
    }, 5000);
</script>
    
{% endblock content %}
{{< /codefile >}}
- *Enregistrer et quitter* 

#### Commentaires sur le code
- Le script {{< focus >}}setTimeout{{< /focus >}} permet à la page de se rafraîchir automatiquement toutes les 5000 millisecondes, de manière à voir les modifications sans avoir à relancer la page
- {{< focus >}}{% extends "_base.html" %}{{< /focus >}} appelle le fichier {{< focus >}}_base.html{{< /focus >}} dans lequel le code de {{< focus >}}index.html{{< /focus >}} va s'insérer
### Avertissement
***Il est possible qu'à ce stade-là, le site affiche des messages d'erreur ; effectivement, dans les fichiers html, notamment le menu, on appelle des pages qui ne sont pas encore construites.***

{{< line >}}
## Les pages de consommation
Il s'agit des pages : 
- jour 
- 24 heures 
- hier 
- dernière heure  
Ces pages affichent la différence de consommation entre un instant {{< focus >}}t_start{{< /focus >}} et un instant {{< focus >}}t_end{{< /focus >}}.
### Les fonctions dans *views.py*
Pour chaque consommation, il convient de faire la différence entre la consommation de début et celle de fin de la période. Ce calcul est donc répétitif et, plutôt que de le copier dans chacune des fonctions, le mieux est de la créer dans un fichier {{< focus >}}utils.py{{< /focus >}} et de l'appeler dans {{< focus >}}views.py{{< /focus >}}
#### Fichier *utils.py*
- Créer le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/utils.py{{< /cmd >}}
- Coller/copier le code suivant :
{{< codefile file="ticServer/ticapp/utils.py" lang="html" >}}
from .models import Data

def formatNumber(nb):
    return '{:.3f}'.format(nb).replace(',', ' ').replace('.', ',')

def conso(dateStart, dateEnd):
    query = Data.objects.filter(dateTime__range=(dateStart, dateEnd)).order_by('id').values("dateTime", "date","east", "easf01", "easf02")
    queryStart = query[0]
    queryEnd = query.reverse()[0]

    kWH = formatNumber((queryEnd["east"] - queryStart["east"])/1000)
    kWHCreux = formatNumber((queryEnd["easf01"] - queryStart["easf01"])/1000)           
    kWHplein = formatNumber((queryEnd["easf02"] - queryStart["easf02"])/1000)
    
    linkyStart = queryStart["date"]
    linkyEnd = queryEnd["date"]

    deltaTime = dateEnd - dateStart
    days = deltaTime.days
    seconds_remaining = deltaTime.seconds
    hours_remaining = seconds_remaining // 3600
    minutes_remaining = (seconds_remaining % 3600) // 60
    seconds_remaining = seconds_remaining % 60
 
    dico = {
        "kWH": kWH,
        "kWHCreux": kWHCreux,
        "kWHPlein": kWHPlein,
        "dateStart": dateStart,
        "dateEnd": dateEnd,
        "linkyStart": linkyStart,
        "linkyEnd": linkyEnd,
        "deltaTime": deltaTime,
        "days": days,
        "hours": hours_remaining,
        "minutes": minutes_remaining,
        "seconds": seconds_remaining,
    }

    return dico
{{< /codefile >}}
- *Sauvegarder et fermer*
#### Commentaires sur le code
- Fonction {{< focus >}}formatNumber{{< /focus >}} : formate un nombre décimal au format français avec un espace comme séparateur de milliers
- {{< focus >}}query{{< /focus >}} : récupère toutes les données entre l'heure de départ et l'heure de fin
- {{< focus >}}queryStart{{< /focus >}} et {{< focus >}}queryEnd{{< /focus >}} : données pour l'heure de départ et l'heure de fin
- Calcul et récupération de diverses données
- Au final, renvoyer un dictionnaire avec les données souhaitées

### Exemple : fonction *jour*
#### Code de la fonction *jour*
Le code de la fonction *jour* est le suivant :
{{< codefile file="ticServer/ticapp/views.py" lang="python" >}}
def jour(request):
    dateEnd = timezone.now()    
    dateStart = timezone.localtime(dateEnd).replace(
    hour=0, minute=0, second=0, microsecond=0)

    values = conso(dateStart, dateEnd)

    context = {'values': values,
    }

    return render(request, "jour.html", context)
{{< /codefile >}}
#### Commentaires
- On utilise la fonction {{< focus >}}conso{{< /focus >}} de {{< focus >}}utils.py{{< /focus >}}
- Il faut donc l'importer dans {{< focus >}}views.py{{< /focus >}} : {{< cmd >}}from utils import conso{{< /cmd >}}
- La manipulation des heures est complexe ; {{< focus >}}dateEnd{{< /focus >}} est une heure avisée ; mais si on remplace les heures, minutes, secondes, microsecondes par zéro (pour être à minuit), l'heure devient naïve et il faut la rendre avisée en utilisant la méthode {{< focus >}}localtime{{< /focus >}}
- Dans d'autres fonctions, on utilisera la méthode {{< focus >}}timedelta{{< /focus >}} de {{< focus >}}datetime{{< /focus >}}, qu'il faut donc importer : {{< cmd >}}from datetime import timedelta{{< /cmd >}}
### Le nouveau fichier *views.py*
- Ouvrir en écriture {{< focus >}}views.py{{< /focus >}} : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/views.py{{< /cmd >}}
- Effacer le code existant
- Le remplacer par le code suivant : 
{{< codefile file="ticServer/ticapp/views.py" lang="python" >}}
from django.shortcuts import render
from django.utils import timezone           
from .models import Data 
from .utils import formatNumber, conso      # new
from datetime import timedelta              # new

def index(request):                        
    now = timezone.now()
    data = Data.objects.last()
    total = format(data.sinsts, ',').replace(',', ' ')
    phase1 = format(data.sinsts1, ',').replace(',', ' ')
    phase2 = format(data.sinsts2, ',').replace(',', ' ')
    phase3 = format(data.sinsts3, ',').replace(',', ' ')
    date = data.date
    creuxplein = data.ltarf
    values = {
        'total': total, 
        'phase1': phase1,
        'phase2': phase2,
        'phase3': phase3,
        'date': date,
        'creuxplein': creuxplein,
    }

    context = {
        'values': values,
        'now': now,
    }

    return render(request, 'index.html', context)

def jour(request):                            # new
    dateEnd = timezone.now()    
    dateStart = timezone.localtime(dateEnd).replace(
    hour=0, minute=0, second=0, microsecond=0)

    values = conso(dateStart, dateEnd)

    context = {'values': values,
    }

    return render(request, "jour.html", context)

def last24hours(request):                   # new
    values = conso(timezone.now() - timedelta(hours = 24), timezone.now())
    context = {'values': values}
    return render(request, "last24hours.html", context)

def hier(request):                          # new   
    
    dateStart = timezone.localtime(timezone.now()).replace(hour=0, minute=0, second=0, microsecond=0) - timedelta(hours=24)
    dateEnd = timezone.localtime(dateStart).replace(hour=23, minute=59, second=59, microsecond=999999)

    values = conso(dateStart, dateEnd)

    context = {'values': values,
    }
    
    return render(request, "hier.html", context)

def lasthour(request):                      # new        
    dateStart = timezone.now() - timedelta(hours = 1)                   
    dateEnd = timezone.now()
    values = conso(dateStart, dateEnd)
    context = {'values': values,
                'dateStart': dateStart,
                'dateEnd': dateEnd,}
    return render(request, "lasthour.html", context)
{{< /codefile >}}

### Créer les *urls*
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/urls.py{{< /cmd >}}
- Entrer les nouvelles *urls* :
{{< codefile file="ticServer/ticapp/urls.py" lang="python" >}}
    path('jour/', views.jour, name='jour'),
    path('lasthour/', views.lasthour, name='lasthour'),
    path('last24hours', views.last24hours, name='last24hours'),    
    path('hier/', views.hier, name='hier'),
{{< /codefile >}}
- Le fichier devient donc celui-ci : 
{{< codefile file="ticServer/ticapp/urls.py" lang="python" >}}
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('jour/', views.jour, name='jour'),
    path('lasthour/', views.lasthour, name='lasthour'),
    path('last24hours', views.last24hours, name='last24hours'),
    path('hier/', views.hier, name='hier'),
]
{{< /codefile >}}
### Créer les gabarits (*templates)*
Pour les 4 fichier html : *jour, lasthour, last24hours, hier*, on va utiliser une présentation commune qui affiche la consommation totale sur la période, la consommation heures pleines, la consommation heures creuses.
Il est donc judicieux de créer un fichier *html* commun qui va être appelé dans chacune des pages qui sera affichée.
#### Fichier *consommation.html*
- Créer en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/consommation.html{{< /cmd >}}
- Copier/coller :
{{< codefile file="ticapp/templates/consommation.html" lang="html" >}}
<div class="flex items-center justify-center pt-2 md:pt-4">
    <table class="border-separate border-spacing-x-8 md:border-spacing-x-16 border-spacing-y-4 md:border-spacing-y-8 border border-gray-200">
        <tbody>
            <tr>
                <td>Total</td>
                <td class="text-right">{{  values.kWH }}</td>
            </tr>
            <tr>
                <td>Heures Creuses</td>
                <td class="text-right">{{  values.kWHCreux }}</td>
            </tr>
            <tr>
                <td>Heures Pleines</td>
                <td class="text-right">{{  values.kWHPlein }}</td>
            </tr>
        </tbody>
    </table>
</div>
{{< /codefile >}}
- *Enregister et fermer*

#### Fichier *jour.html*
- Créer en écriture : {{< cmd>}}nano ~/djangoTIC/ticServer/ticapp/templates/jour.html{{< /cmd >}}
- Copier/coller :
{{< codefile file="ticapp/templates/jour.html" lang="html" >}}
{% extends "_base.html" %}
{% load l10n %}
{% load tz %}

{% block content %}

<div class="max-w-3xl mx-auto text-center mt-12">
    <h1 class="text-xl md:text-3xl lg:text-4xl font-bold text-gray-700 leading-tight mb-2 border-b-2 border-gray-400 pb-2">
      Consommation du Jour (en kWh)
    </h1>
</div>

{% timezone "Europe/Paris" %}
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-semibold text-gray-500">{{ values.dateEnd | date:" D j F Y "}} de minuit à {{ values.dateEnd | date:" H:i "}} </p>
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-medium text-gray-500">Horaires LINKY : {{ values.linkyStart }} / {{ values.linkyEnd }} </p>
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-medium text-gray-500">Durée : {{ values.hours }} heures {{ values.minutes }} minutes {{ values.seconds }} secondes</p>
</div>

{% endtimezone %}

{% localize on %}
{% include "consommation.html" %}
{% endlocalize %}
{{< /codefile >}}
- *Enregister et fermer*

#### Fichier *lasthour.html*
- Créer en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/lasthour.html{{< /cmd >}}
- Copier/coller :
{{< codefile file="ticapp/templates/lasthour.html" lang="html" >}}
{% extends "_base.html" %}

{% block content %}
<div class="max-w-3xl mx-auto text-center mt-12">
    <h1 class="text-xl md:text-3xl lg:text-4xl font-bold text-gray-700 leading-tight mb-2 border-b-2 border-gray-400 pb-2">
      Dernière heure (en kWh)
    </h1>
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-semibold text-gray-500">{{ dateStart | date:" D j F Y "}} de {{ dateStart | date:" H:i:s "}} à {{ dateEnd | date:" H:i:s "}} </p>
</div>

{% include "consommation.html" %}

{% endblock content %}
{{< /codefile >}}
- *Enregister et fermer*

#### Fichier *last24hours.html*
- Créer en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/last24hours.html{{< /cmd >}}
- Copier/coller :
{{< codefile file="ticapp/templates/last24hours.html" lang="html" >}}
{% extends "_base.html" %}

{% load l10n %}
{% load tz %}

{% block content %}

{% timezone "Europe/Paris" %}

<div class="max-w-3xl mx-auto text-center mt-12">
    <h1 class="text-xl md:text-3xl lg:text-4xl font-bold text-gray-700 leading-tight mb-2 border-b-2 border-gray-400 pb-2">
      Dernières 24 heures (en kWh)
    </h1>
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-semibold text-gray-500">
        Heure : {{ values.dateEnd | date:"H:i:s"}}</p>
        
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p>du {{ values.dateStart | date:"l j F Y"}} au 
        {{ values.dateEnd | date:"l j F Y"}} </p>
</div>

<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-semibold text-gray-500">Horaires LINKY : {{ values.linkyStart }} / {{ values.linkyEnd }} </p>
</div>

{% endtimezone %}

{% include "consommation.html" %}

{% endblock content %}
{{< /codefile >}}
- *Enregister et fermer*

#### Fichier *hier.html*
- Créer en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/hier.html{{< /cmd >}}
- Copier/coller :
{{< codefile file="ticapp/templates/hier.html" lang="html" >}}
{% extends "_base.html" %}
{% load l10n %}
{% load tz %}

{% block content %}
<div class="max-w-3xl mx-auto text-center mt-12">
    <h1 class="text-xl md:text-3xl lg:text-4xl font-bold text-gray-700 leading-tight mb-2 border-b-2 border-gray-400 pb-2">
      Consommation de la Veille (en kWh)
    </h1>
</div>

{% timezone "Europe/Paris" %}

<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-semibold text-gray-500">{{ values.dateStart | date:" l j F Y "}} <br><br>de minuit à 23h59mn59s </p>
</div>
<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-medium text-gray-500">Horaires LINKY : {{ values.linkyStart }} / {{ values.linkyEnd }} </p>
</div>

{% endtimezone %}

{% include "consommation.html" %}

{% endblock content %}
{{< /codefile >}}
- *Enregister et fermer*

## Intégrer le menu
Tous les fichiers *html*, toutes les *urls*, toutes les fonctions de *views.py* étant fonctionnels, on peut intégrer le menu dans {{< focus >}}_base.html{{< /focus >}}.
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/_base.html{{< /cmd >}}
- Ajouter {{< cmd >}}{% include 'menu.html' %}{{< /cmd >}}tout en haut du {{< focus >}}body{{< /focus >}}
- Sauvegarder et fermer
- Le fichier complet est le suivant : 
{{< codefile file="ticapp/templates/_base.html" lang="html" >}}
{% load static %}

<!DOCTYPE html>
<html lang="fr">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>TIC Server</title>
        <link rel="stylesheet" href="{% static 'src/output.css' %}">
    </head>
    <body class="bg-green-50">   
        {% include 'menu.html' %}
        <div class="container mx-auto mt-4">
            {% block content %}
            {% endblock content %}
        </div>
        <script src="{% static 'flowbite/dist/flowbite.min.js' %}"></script>       
    </body>
</html>
{{< /codefile >}}