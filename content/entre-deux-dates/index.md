+++
date = '2026-01-22T18:01:54+01:00'
draft = false
title = 'Consommation entre deux dates'
description = "Lire la consommation du compteur Linky entre deux dates séléctionnées par le date-picker de flowbite ; site web Django installé sur Raspberry Pi."
+++

## Objectif et Moyens
- L'objectif est d'afficher, sur une page web spécifique, la consommation électrique entre deux dates déterminées
- Pour sélectionner les dates on utilise le sélecteur de date [*Date-picker* de Flowbite](https://flowbite.com/docs/components/datepicker/)
- Pour cela il faut
  - créer une fonction {{< focus >}}entre2dates{{< /focus >}} dans *ticapp/views.py* 
  - créer une *url* spécifique dans *ticapp/urls.py*
  - créer un gabarit *(template)* {{< focus >}}entre2dates.html{{< /focus >}} dans *ticapp/templates*
- C'est dans ce gabarit que le sélecteur de date apparaîtra en tant que formulaire remplissable

## Préalable
- La mode *debug* de *Django* a été désactivé (voir le tuto [Mise en Production]({{< relref "tuto//part6_mise-en-production#passer-en-production" >}}))
- Il peut être intéressant d'avoir les messages de débogage fournis par Django lors de l'installation de nouvelles pages
- Pour activer le mode *debug* :
  - ouvrir le fichier *.env* : {{< cmd>}}nano ~/djangoTIC/ticServer/.env{{< /cmd >}}
  - fixer la valeur : {{< focus >}}DEBUG=True{{< /focus >}}
  - relancer le serveur : {{< cmd >}}sudo systemctl restart django-ticserver{{< /cmd >}}
- Ce préalable n'est pas obligé
## Fonctionnement du sélecteur de date (datepicker)
### Récupération de la date
- Le sélecteur de date est inséré dans une balise [form](https://developer.mozilla.org/fr/docs/Web/HTML/Reference/Elements/form) du gabarit *entredeuxdates.html*
- Les dates sont saisies dans des balises [input](https://developer.mozilla.org/fr/docs/Web/HTML/Reference/Elements/input)
- L'attribut {{< focus >}}class="datepicker"{{< /focus >}} permet à *Flowbite* de détecter cet *input* et d'y attacher automatiquement son composant de sélection de date
- La valeur renvoyée par *l'input* est une chaîne de caractères ([string](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Global_Objects/String)) représentant la date
- Par défaut, le *datepicker de Flowbite* renvoie une date au format américain {{< focus >}}mm/dd/yyyy{{< /focus >}}
- Pour faciliter le traitement côté serveur Django, nous ajoutons l'attribut {{< focus >}}datepicker-format="dd/mm/yyyy"{{< /focus >}} afin d'obtenir une date au format européen {{< focus >}}dd/mm/yyyy{{< /focus >}}, exploitable par nos fonctions *Python*
### Traduction du sélecteur de date
- Par défaut, le *datepicker de Flowbite* s'affiche en anglais
- La bibliothèque *Flowbite* ne propose pas de solution native pour la traduction en français
- Nous utilisons donc un script de configuration adapté d'une solution proposée par [@gritvald pour le polonais](https://github.com/themesberg/flowbite/issues/32#issuecomment-2402324109), que nous avons francisé pour nos besoins
### Où placer le script de traduction ?
- De manière à pouvoir réutiliser ce script dans d'autres pages que *entredeuxdates.html*, on place le script dans un dossier *ticapp/static/js*
- Créer le dossier : {{< cmd >}}mkdir ~/djangoTIC/ticServer/ticapp/static/js{{< /cmd >}}
- Créer et ouvrire le ficher en écriture {{< focus >}}datepicker-fr.js{{< /focus >}} : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/static/js/datepicker-fr.js{{< /cmd >}}
- Copier/coller le code :
{{< codefile file="static/js/datepicker-fr.js" lang="js" >}}
window.addEventListener("load", () => {
    setTimeout(() => {
        let locales = {
            fr: {
                days: ["Dimanche", "Lundi", "Mardi", "Mercredi", "Jeudi", "Vendredi", "Samedi"],
                daysShort: ["Dim", "Lun", "Mar", "Mer", "Jeu", "Ven", "Sam"],
                daysMin: ["Di", "Lu", "Ma", "Me", "Je", "Ve", "Sa"],
                months: ["Janvier", "Février", "Mars", "Avril", "Mai", "Juin", "Juillet", "Août", "Septembre", "Octobre", "Novembre", "Décembre"],
                monthsShort: ["Janv.", "Févr.", "Mars", "Avr.", "Mai", "Juin", "Juil.", "Août", "Sept.", "Oct.", "Nov.", "Déc."],
                today: "Aujourd'hui",
                weekStart: 1,
                clear: "Effacer",
                format: "dd/mm/yyyy"
            }
        };
        let flowbitePickers = Object.values(FlowbiteInstances.getInstances("Datepicker")).map((instance) => {
            return instance.getDatepickerInstance();
        });
        for (const flowbitePicker of flowbitePickers) {
            for (const picker of flowbitePicker.datepickers || [flowbitePicker]) {
                Object.assign(picker.constructor.locales, locales);
                picker.setOptions({ language: "fr" });
                picker.setDate({ clear: true }); // Vider aussi l'instance du datepicker
            }
        }
    }, 100);
});
{{< /codefile >}}
- Sauvegarder et quitter

### Comment appeler le script dans le html ?
#### Ordre de chargement des scripts
- Pour fonctionner correctement, le script de traduction *datepicker-fr.js* doit être chargé **après** *Flowbite*
- En effet, il doit pouvoir accéder aux composants *Flowbite* déjà initialisés pour les configurer en français
#### Problématique d'architecture
- Dans notre *template* de base *_base.html*, le script *Flowbite* est appelé juste avant la balise de fermeture *&lt;/body&gt;* :  
{{< cmdNocopyGoLine >}}&lt;script src="&#123;% static 'flowbite/dist/flowbite.min.js' %&#125;"&gt;&lt;/script&gt;  
&lt;/body&gt; 
{{< /cmdNocopyGoLine >}}
- Si nous placions l'appel à *datepicker-fr.js* à la fin du bloc *{% block content %}* de *entre2dates.html*, celui-ci serait en réalité chargé avant *Flowbite*, ce qui poserait problème
- De plus, ce script n'a pas besoin d'être appelé dans chaque page, mais uniquement dans celles qui utilisent le *datepicker* de *Flowbite*
#### Solution : créer un bloc dédié aux scripts
- Nous créons un nouveau bloc {{<focus >}}{% block scripts %}{{< /focus >}} dans *_base.html*, positionné après l'appel de *Flowbite* :  
{{< cmdNocopyGoLine >}}&lt;script src="&#123;% static 'flowbite/dist/flowbite.min.js' %&#125;"&gt;&lt;/script&gt;  
{% block scripts %}{% endblock %}  
&lt;/body&gt;
{{< /cmdNocopyGoLine >}}
- Ainsi, dans les pages contenant un datepicker, nous pouvons surcharger ce bloc pour ajouter notre script de traduction :  
{{< cmdNocopyGoLine >}}{% block scripts %}  
&lt;script src="&#123;% static 'js/datepicker-fr.js' %&#125;"&gt;&lt;/script&gt;   
{% endblock %}
{{< /cmdNocopyGoLine >}}
### Traiter les données dans *views.py*
Le traitement des dates saisies, pour permettre l'affichage des données, s'effectue dans la fonction {{< focus >}}entre2dates{{< /focus >}}de *ticapp/views.py*
#### Récupération des données du formulaire
- La méthode GET permet de récupérer les valeurs des champs de date via leurs identifiants {{< focus >}}start{{< /focus >}} et {{< focus >}}end{{< /focus >}} :  
{{< cmdNocopyGoLine >}}date_debut_str = request.GET.get('start')  
date_fin_str = request.GET.get('end'){{< /cmdNocopyGoLine >}}
#### Conversion et normalisation des dates
Les dates reçues sous forme de chaînes de caractères nécessitent plusieurs transformations :
- Conversion en objets {{< focus >}}datetime{{< /focus >}} à partir du format *dd/mm/yyyy*
- Définition des heures de début et de fin de journée : nous utilisons {{< focus >}}combine(){{< /focus >}} avec {{< focus >}}datetime.min.time(){{< /focus >}} *(00:00:00)* pour la date de début et {{< focus >}}datetime.max.time(){{< /focus >}} *(23:59:59.999999)* pour la date de fin
- Conversion en dates [avisées](https://www.docstring.fr/blog/la-gestion-des-dates-avec-python/#:~:text=Une%20date%20%C2%AB%20naive%20%C2%BB%20est%20une,selon%20un%20fuseau%20horaire%20pr%C3%A9cis.) (conscientes du fuseau horaire) pour garantir la cohérence avec les données du compteur Linky
#### Calculs de consommation
- La fonction *conso* du module *ticapp/utils.py* traite les données en fonction des dates définies plus haut
- Pour avoir la consommation d'un jour unique, il suffit de sélectionner la même date de début et de fin : la définition des heures couvre le nycthémère entier
- La fonction *conso* renvoie des kWh au format décimal ; la lecture peut être malaisée si les valeurs sont importantes
- On va arrondir ces valeurs dans la fonction *entre2dates* pour afficher les arrondis dans le gabarit *entre2dates.html*

## Les différents fichiers
### *ticapp/urls.py*
- Ouvrir en écriture *ticapp/urls.py* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/urls.py{{< /cmd >}}
- Ajouter l'*url* de la page à la fin de la liste *urlpatterns* : {{< cmd >}}path('entre2dates/', views.entre2dates, name='entre2dates'),{{< /cmd >}}
- Ici, la valeur "name" a toute son importance, car c'est elle qui va être appelée dans l'attribut {{< focus >}}action{{< /focus >}} du formulaire *form*
- Le fichier *ticapp/urls.py* ressemble à celui-ci : 
{{< codefile file="ticServer/ticapp/urls.py" lang="python" >}}
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('jour/', views.jour, name='jour'),
    path('lasthour/', views.lasthour, name='lasthour'),
    path('last24hours', views.last24hours, name='last24hours'),
    path('hier/', views.hier, name='hier'),
    path('entre2dates/', views.entre2dates, name='entre2dates'),
]
{{< /codefile >}}

### *ticapp/views.py*
- On ajoute la fonction {{< focus >}}entre2dates{{< /focus >}} dans le fichier *views.py*
- Comme on utilise la méthode {{< focus >}}datetime.datetime{{< /focus >}} du module *datetime*, il faut s'assurer de son import : {{< cmd >}}from datetime import datetime{{< /focus >}}
- Le code de la fonction est le suivant :
{{< codefile file="views.py/entre2dates" lang="python" >}}
def entre2dates(request):
    """on récupère les strings du formulaire
    Afin de ne pas afficher la div du html tant que les dates ne sont pas choisies :
    on initialise des variables à None
    et on ne traite que si les deux dates sont présentes
    """
    start = request.GET.get('start')	# récupération de l'input start
    end = request.GET.get('end')	# récupération de l'input end

    values = None			# initialisation de values
    
    if start and end:   # si la requête GET a renvoyé des valeurs 
        try:
            # Convertir les dates

            start_date = datetime.strptime(start, '%d/%m/%Y')  # format français du date picker
            end_date = datetime.strptime(end, '%d/%m/%Y')      # format français du date picker
            
            # Début du jour start (00:00:00)
            start_datetime = timezone.make_aware(datetime.combine(start_date.date(), datetime.min.time()))
            
            # Fin du jour end (23:59:59) ; utilisation de combine() ; rendre la date avisée : make_aware()
            end_datetime = timezone.make_aware(datetime.combine(end_date.date(), datetime.max.time()))
            
            # Récupérer les valeurs avec la fonction conso
            values = conso(start_datetime, end_datetime)
            
        except ValueError:
            # IndexError peut arriver si query[0] est vide (pas de données en base)
            values = None       

    context = {
        "values": values,
        "start": start, 	# INDISPENSABLE pour le {% if start and end %}
        "end": end,     	# INDISPENSABLE pour le {% if start and end %}
        # pour afficher les consommations en entiers
        # conso renvoie des valeurs avec une virgule pour les nombres décimaux
        # remplacer la virgule par un point pour rendre la valeur lisible par float()
        "roundkWH": round(float(values['kWH'].replace(",", "."))) if values else None,
        "roundkWHCreux": round(float(values['kWHCreux'].replace(",", "."))) if values else None,
        "roundkWHPlein": round(float(values['kWHPlein'].replace(",", "."))) if values else None,
    }

    return render(request, 'entre2dates.html', context)	# renvoi du contexte{{< /codefile >}}
- Ouvrir le fichier *views.py* en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/views.py{{< /cmd >}}
- Ajouter le code de la fonction à la fin du fichier *views.py*
- Le fichier *views.py* ressemble à ceci :  
{{< codefile file="ticServer/ticapp/views.py" lang="python" >}}
from django.shortcuts import render
from django.utils import timezone           
from .models import Data 
from .utils import formatNumber, conso 
from datetime import timedelta, datetime

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

def jour(request): 
    dateEnd = timezone.now()    
    dateStart = timezone.localtime(dateEnd).replace(
    hour=0, minute=0, second=0, microsecond=0)

    values = conso(dateStart, dateEnd)

    context = {'values': values,
    }

    return render(request, "jour.html", context)

def last24hours(request): 
    values = conso(timezone.now() - timedelta(hours = 24), timezone.now())
    context = {'values': values}
    return render(request, "last24hours.html", context)

def hier(request):    
    dateStart = timezone.localtime(timezone.now()).replace(hour=0, minute=0, second=0, microsecond=0) - timedelta(hours=24)
    dateEnd = timezone.localtime(dateStart).replace(hour=23, minute=59, second=59, microsecond=999999)

    values = conso(dateStart, dateEnd)

    context = {'values': values,
    }
    
    return render(request, "hier.html", context)

def lasthour(request):        
    dateStart = timezone.now() - timedelta(hours = 1)                   
    dateEnd = timezone.now()
    values = conso(dateStart, dateEnd)
    context = {'values': values,
                'dateStart': dateStart,
                'dateEnd': dateEnd,}
    return render(request, "lasthour.html", context)
    
def entre2dates(request):
    start = request.GET.get('start')
    end = request.GET.get('end')

    values = None
    
    if start and end:
        try:
            start_date = datetime.strptime(start, '%d/%m/%Y') 
            end_date = datetime.strptime(end, '%d/%m/%Y') 

            start_datetime = timezone.make_aware(datetime.combine(start_date.date(), datetime.min.time()))
            end_datetime = timezone.make_aware(datetime.combine(end_date.date(), datetime.max.time()))
            
            values = conso(start_datetime, end_datetime)
            
        except ValueError:
            values = None       

    context = {
        "values": values,
        "start": start, 
        "end": end,
        "roundkWH": round(float(values['kWH'].replace(",", "."))) if values else None,
        "roundkWHCreux": round(float(values['kWHCreux'].replace(",", "."))) if values else None,
        "roundkWHPlein": round(float(values['kWHPlein'].replace(",", "."))) if values else None,
    }

    return render(request, 'entre2dates.html', context)
{{< /codefile >}}

### *ticapp/templates/_base.html*
- Le fichier doit être modifié pour insérer le nouveau *block scripts*
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/_base.html{{< /cmd >}}
- Ajouter **après** l'appel du script *flowbite* l'appel de bloc :  
{{< cmdNocopyGoLine >}}{% block scripts %}
{% endblock %}{{< /cmdNocopyGoLine >}}
- Le fichier *_base.html* ressemble à : 
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

    {% block scripts %}
    {% endblock scripts %}
    
</body>
</html>
{{< /codefile >}}

### *ticapp/templates/entre2dates.html*
#### Commentaires
- En haut de la page les deux sélecteurs de date vont s'afficher dans une balise *\<form\>*
- La balise *\<div\>* contenant les résultats ne s'affiche que si les deux dates ont été saisies : elle est donc soumise à une condition {{< focus >}}{% if start and end %}{{< /focus >}}
- On affiche les valeurs avec décimales et arrondies
#### Le fichier
- Créer et ouvrir en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/entre2dates.html{{< /cmd >}}
- Copier/coller le code :
{{< codefile file="ticapp/templates/entre2dates.html" lang="html" >}}
{% extends "_base.html" %}
{% load static %} 

{% load l10n %}
{% load tz %}

{% block content %}

<div class="max-w-3xl mx-auto text-center mt-12">
    <h1 class="text-xl md:text-3xl lg:text-4xl font-bold text-gray-700 leading-tight mb-2 border-b-2 border-gray-400 pb-2">
      Entre deux dates (en kWh)
    </h1>
</div>

<div class="flex justify-center w-full mt-4 md:mt-8">
    <form method="GET" action="{% url 'entre2dates' %}" class="flex items-center">
        <div id="date-range-picker" date-rangepicker class="flex items-center ">
            <div class="relative">
                <div class="absolute inset-y-0 start-0 flex items-center ps-3 pointer-events-none">
                    <svg class="w-4 h-4 text-body" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24"><path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 10h16m-8-3V4M7 7V4m10 3V4M5 20h14a1 1 0 0 0 1-1V7a1 1 0 0 0-1-1H5a1 1 0 0 0-1 1v12a1 1 0 0 0 1 1Zm3-7h.01v.01H8V13Zm4 0h.01v.01H12V13Zm4 0h.01v.01H16V13Zm-8 4h.01v.01H8V17Zm4 0h.01v.01H12V17Zm4 0h.01v.01H16V17Z"/></svg>
                </div>
                <input id="start"name="start" type="text" autocomplete="off" datepicker datepicker-format="dd/mm/yyyy" class="bg-green-100 border border-green-200 text-gray-900 text-sm rounded-lg focus:ring-green-500 focus:border-green-500 block w-full ps-10 p-2.5" placeholder="Date de début">

            </div>
            <span class="mx-8 text-gray-500">à</span>
            <div class="relative">
                <div class="absolute inset-y-0 start-0 flex items-center ps-3 pointer-events-none">
                    <svg class="w-4 h-4 text-body" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="none" viewBox="0 0 24 24"><path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 10h16m-8-3V4M7 7V4m10 3V4M5 20h14a1 1 0 0 0 1-1V7a1 1 0 0 0-1-1H5a1 1 0 0 0-1 1v12a1 1 0 0 0 1 1Zm3-7h.01v.01H8V13Zm4 0h.01v.01H12V13Zm4 0h.01v.01H16V13Zm-8 4h.01v.01H8V17Zm4 0h.01v.01H12V17Zm4 0h.01v.01H16V17Z"/></svg>
                </div>
                <input id="end"name="end" type="text" autocomplete="off"datepicker datepicker-format="dd/mm/yyyy" class="bg-green-100 border border-green-200 text-gray-900 text-sm rounded-lg focus:ring-green-500 focus:border-green-500 block w-full ps-10 p-2.5" placeholder="Date de fin">
            </div>
        </div>
        <button type="submit" class="ms-8 px-2 text-white bg-gray-700 rounded-lg border border-gray-800 hover:bg-gray-800 hover:border-gray-900 ">
            Filtrer
        </button>
    </form>
</div>

{% if start and end %}

    <div class="flex items-center justify-center pt-2 md:pt-4">
        <p>du {{ values.dateStart | date:"l j F Y"}} au 
            {{ values.dateEnd | date:"l j F Y"}} </p>
    </div>

    {% include "consommation.html"  %}

    <div class="flex items-center justify-center pt-2 md:pt-6">
        <p class="font-bold">En valeurs arrondies (kWh)</p>
    </div>
    <div class="flex items-center justify-center pt-2 md:pt-4">

    <table class="border-separate border-spacing-x-8 md:border-spacing-x-16 border-spacing-y-4 md:border-spacing-y-8 border border-gray-200">
        <tbody>
            <tr>
                <td>Total</td>
                <td class="text-right">{{  roundkWH }}</td>
            </tr>
            <tr>
                <td>Heures Creuses</td>
                <td class="text-right">{{  roundkWHCreux }}</td>
            </tr>
            <tr>
                <td>Heures Pleines</td>
                <td class="text-right">{{  roundkWHPlein }}</td>
            </tr>
        </tbody>
    </table>
</div>

{% endif %}
{% endblock content %}

{% block scripts %}

 <script src="{% static 'js/datepicker-fr.js' %}"></script>

{% endblock scripts %}
{{< /codefile >}}
### *ticapp/templates/menu.html*
- On a ajouté une nouvelle page : il faut faire apparaître celle-ci dans le menu
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/menu.html{{< /cmd >}}
- Ajouter le contenu juste avant la femerture de la balise : *\</ul\>*
{{< codefile file="ticapp/templates/menu.html" lang="html" >}}<li>
<a href="{% url 'entre2dates' %}" class="block py-2 pl-3 pr-4 text-gray-700 hover:bg-gray-50 md:hover:bg-transparent md:border-0 md:hover:text-green-700 md:p-0 dark:text-gray-400 md:dark:hover:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent">Par dates</a>
</li>
{{< /codefile >}}
- Le code final du fichier est :
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
        <li>
            <a href="{% url 'entre2dates' %}" class="block py-2 pl-3 pr-4 text-gray-700 hover:bg-gray-50 md:hover:bg-transparent md:border-0 md:hover:text-green-700 md:p-0 dark:text-gray-400 md:dark:hover:text-white dark:hover:bg-gray-700 dark:hover:text-white md:dark:hover:bg-transparent">Par dates</a>
        </li>            
        </ul>
    </div>
    </div>
</nav>
{{< /codefile >}}

## Finalisation
### Fichier *.env*
- Si on a mis la valeur *DEBUG* à *True*, la remettre à *False*
- Ouvrir le fihcier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/.env{{< /cmd >}}
- Fixer la valeur : {{< focus >}}DEBUG=False{{< /focus >}}

### Actualiser le css 
- Le *css* a été modifié : il faut l'actualiser
- Se mettre en environnement virtuel : {{<cmd >}}source ~/djangoTIC/venv/bin/activate{{< /cmd >}}
- Se positionner dans le répertoire *djangoTIC/ticServer* : {{< cmd >}}cd ~/djangoTIC/ticServer{{< /cmd >}}
- Mettre à jour le fichier *src/ouput.css* : {{< cmd >}}npx @tailwindcss/cli -i ticapp/static/src/input.css -o ticapp/static/src/output.css \-\-minify{{< /cmd >}}
### Collecter les fichiers *static*
- Dans la mesure où on a modifié le *css* et créé un fichier statique *js/datepicker-fr.js*, il faut collecter les fichiers statiques pour les rendre lisibles par *gunicorn*
- Rester en environnement virtuel dans le dossier *djangoTIC*ticServer*
- Lancer la commande : {{< cmd >}}python manage.py collectstatic \-\-noinput{{< /cmd >}}
- Relancer le serveur : {{< cmd >}}sudo systemctl restart django-ticserver{{< /cmd >}}
### Sélecteur horaire
- Si on le souhaite on peut affiner les chiffres en ajoutant un sélecteur horaire, mais ceci a peu d'intérêt pour un affichage de consommation électrique, tel que celui proposé dans ce tutoriel
- *Fowbite* possède une fonction [timepicker](https://flowbite.com/docs/forms/timepicker/)
### Mise en garde
Par souci de simplicité (et, soyons honnête, un peu de paresse), ce tutoriel n'implémente pas de gestion d'erreurs complète. En production, il serait judicieux d'ajouter des validations de formulaire et des blocs {{< focus >}}try...except{{< /focus >}}.
Soyez donc particulièrement vigilant lors de la saisie des dates : un format incorrect ou des dates incohérentes généreront des erreurs non gérées.
