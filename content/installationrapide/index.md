+++
date = '2025-12-19T18:19:10+01:00'
draft = false
title = 'Installation Rapide'
description = "Installation rapide d'un site django sur une raspberry pour lire les données d'un compteur Linky ; utilisation de picocom (téléinformation client) et  unicorn (serveur)"
+++
{{< line >}}
Ce tutoriel permet d'installer rapidement les éléments nécessaires sur la Raspberry.  
On présuppose que :
- la Raspberry est fonctionnelle et mise à jour
- la Raspberry est accessible en ssh
- elle est dotée d'une adresse IP locale fixe
- le module PiTinfo est installé  
- la *téléinformation client* est en mode *standard* (et non *historique*)  
Les commandes sont lancées dans un terminal de la Raspberry.  
Ce tutoriel est valide pour une installation en triphasé avec horaires jour/nuit. Les codes doivent être adaptés dans une configuration différente.
{{< line >}}
## Installation des paquets et logiciels
- Installer les différents paquets : {{< cmd >}}sudo apt install picocom nodejs npm{{< /cmd >}}
- Ouvrir le fichier */boot/firmware/config.txt* : {{< cmd >}}sudo nano /boot/firmware/config.txt{{< /cmd >}}
- Effacer son contenu
- Le remplacer par : 
{{< codefile file="/boot/firmware/config.txt" lang="txt" >}}
# For more options and information see
# http://rptl.io/configtxt
# Some settings may impact device functionality. See link above for details

# Uncomment some or all of these to enable the optional hardware interfaces
#dtparam=i2c_arm=on
#dtparam=i2s=on
#dtparam=spi=on

# Enable audio (loads snd_bcm2835)
dtparam=audio=on

# Additional overlays and parameters are documented
# /boot/firmware/overlays/README

# Automatically load overlays for detected cameras
camera_auto_detect=1

# Automatically load overlays for detected DSI displays
display_auto_detect=1

# Automatically load initramfs files, if found
auto_initramfs=1

# Enable DRM VC4 V3D driver
dtoverlay=vc4-kms-v3d
max_framebuffers=2

# Don't have the firmware create an initial video= setting in cmdline.txt.
# Use the kernel's default instead.
disable_fw_kms_setup=1

# Run in 64-bit mode
arm_64bit=1

# Disable compensation for displays with overscan
disable_overscan=1

# Run as fast as firmware / board allows
arm_boost=1

[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
otg_mode=1

[cm5]
dtoverlay=dwc2,dr_mode=host

[all]
#enable_uart=0
enable_uart=1

#ajout pour lecture linky
dtoverlay=disable-bt
{{< /codefile >}}
- Créer le dossier *djangoTIC* : {{< cmd >}}mkdir -p ~/djangoTIC {{< /cmd >}}
- Créer l'environnement virtuel : {{< cmd >}}python3 -m venv ~/djangoTIC/venv{{< /cmd >}}
- Se mettre en environnement virtuel : {{< cmd >}}source ~/django/venv/bin/activate{{< /cmd >}}
- Installer *Django* : {{< cmd >}}python -m pip install django{{< /cmd >}}
- Aller dans *~/djangoTIC* : {{< cmd >}}cd ~/djangoTIC{{< /cmd >}}
- Créer le projet *ticServer* : {{< cmd >}}django-admin startproject ticServer
{{< /cmd >}}
- Aller dans *~/djangoTIC/ticServer* : {{< cmd >}}cd ~/djangoTIC/ticServer{{< /cmd >}}
- Créer l'application *ticapp* : {{< cmd >}}python manage.py startapp ticapp{{< /cmd >}}
- Créer les différents dossiers : {{< cmd >}}mkdir -p ~/djangoTIC/ticServer/ticapp/templates ~/djangoTIC/ticServer/ticapp/management/commands ~/djangoTIC/ticServer/ticapp/static/src ~/djangoTIC/ticServer/ticapp/static/images{{< /cmd >}}
- Initialiser les modules python :  
{{< cmd >}}touch djangoTIC/ticServer/ticapp/management/__init__.py{{< /cmd >}}  
{{< cmd >}}touch djangoTIC/ticServer/ticapp/management/commands/__init__.py{{< /cmd >}}
- Installer *tailwind* et *flowbite* : {{< cmd >}}npm install tailwindcss @tailwindcss/cli flowbite{{< /cmd >}}
- Copier *flowbite* vers *static* : {{< cmd >}}cp -r ~djangoTIC/ticServer/node_modules/flowbite ~djangoTIC/ticServer/ticapp/static/{{< /cmd >}}
- Créer le fichier *input.css* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/static/src/input.css{{< /cmd >}}
- Copier/coller le contenu :
{{< codefile file="static/src/input.css" lang="css" >}}
@import "tailwindcss";
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
@import "flowbite/src/themes/default";
@plugin "flowbite/plugin";
@source "../flowbite";
{{< /codefile >}}
- Installer *pyserial* : {{< cmd >}}pip install pyserial{{< /cmd >}}
- Installer *gunicorn* : {{< cmd >}}pip install gunicorn{{< /cmd >}}
- Installer *whitenoise* : {{< cmd >}}pip install whitenoise{{< /cmd >}}
- Installer *python-decouple* : {{< cmd >}}pip install python-decouple{{< /cmd >}}
## Création du site
### models.py - views.py - urls.py
- Modifier *models.py* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/models.py{{< /cmd >}}
- Effacer le contenu puis copier/coller : 
{{< codefile file="ticServer/ticapp/models.py" lang="python" >}}
from django.db import models

class Data(models.Model):

    dateTime = models.DateTimeField(auto_now_add=True, db_index=True)   # la date et l'heure données par l'ordinateur
    date = models.CharField(max_length=13,blank=True, null=True)        # la date et l'heure données par le TIC
    ltarf = models.CharField(max_length=16,blank=True, null=True)       # Type de tarif en cours
    east = models.IntegerField(blank=True, null=True)                   # Energie active soutirée Fournisseur Consommation totale
    easf01 = models.IntegerField(blank=True, null=True)                 # Energie active soutirée Fournisseur index 1 Consommation heures creuses
    easf02 = models.IntegerField(blank=True, null=True)                 # Energie active soutirée Fournisseur index 2 Consommation heures pleines
    easd01 = models.IntegerField(blank=True, null=True)                 # Energie active soutirée Distributeur index 1
    easd02 = models.IntegerField(blank=True, null=True)                 # Energie active soutirée Distributeur index 2   
    easd03 = models.IntegerField(blank=True, null=True)                 # Energie active soutirée Distributeur index 3
    easd04 = models.IntegerField(blank=True, null=True)                 # Energie active soutirée Distributeur index 4
    sinsts = models.IntegerField(blank=True, null=True)                 # Puissance apparente instantanée soutirée 
    sinsts1 = models.IntegerField(blank=True, null=True)                # Puissance apparente instantanée soutirée phase 1
    sinsts2 = models.IntegerField(blank=True, null=True)                # Puissance apparente instantanée soutirée phase 2
    sinsts3 = models.IntegerField(blank=True, null=True)                # Puissance apparente 
{{< /codefile >}}
- Créer le fichier *utils.py* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/utils.py{{< /cmd >}}
- Copier/coller le contenu :
{{< codefile file="ticServer/ticapp/utils.py" lang="python" >}}
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
- Modifier *views.py* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/views.py{{< /cmd >}}
- Effacer le contenu puis copier/coller :
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
- Créer *ticapp/utls.py* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/urls.py{{< /cmd >}}
- Copier/coller le contenu : 
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
- Modifier le fichier *ticServer/urls.py* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticServer/urls.py{{< /cmd >}}
- Effacer le contenu et le remplacer par : 
{{< codefile file="ticServer/ticServer/urls.py" lang="python" >}}
from django.contrib import admin
from django.urls import path, include   # include : new

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('ticapp.urls')),   # new    
]
{{< /codefile >}}
### Gabarits (templates)
- Créer le gabarit *_base.html* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/_base.html{{< /cmd >}}
- Copier/coller le contenu : 
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
- Créer le gabarit *menu.html* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/menu.html{{< /cmd >}}
- Copier/coller le contenu : 
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
- Télécharger l'image "linkylogo.png" : {{< cmd >}}wget -O ~/djangoTIC/ticServer/ticapp/static/images/linkylogo.png https://linky-tuto.netlify.app/images/linkylogo.png{{< /cmd >}}
- Créer le gabarit *index.html* : {{< cmd >}}nano djangoTIC/ticServer/ticapp/templates/index.html{{< /cmd >}}
- Copier/coller le contenu : 
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
- Créer le gabarit *consommation.html* : {{< cmd >}}nano djangoTIC/ticServer/ticapp/templates/consommation.html{{< /cmd >}}
- Copier/coller le contenu : 
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
- Créer le gabarit *jour.html* : {{< cmd >}}nano djangoTIC/ticServer/ticapp/templates/jour.html{{< /cmd >}}
- Copier/coller le contenu : 
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
- Créer le gabarit *lasthour.html* : {{< cmd >}}nano djangoTIC/ticServer/ticapp/templates/lasthour.html{{< /cmd >}}
- Copier/coller le contenu : 
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
- Créer le gabarit *last24hours.html* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/last24hours.html{{< /cmd >}}
- Copier/coller le contenu : 
{{< codefile file="ticapp/templates/last24thours.html" lang="html" >}}
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
- Créer le gabarit *hier.html* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/hier.html{{< /cmd >}}
- Copier/coller le contenu : 
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
## Commandes et services
- Créer la commande *capture_tic* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/management/commands/capture_tic.py{{< /cmd >}}
- Copier/coller le contenu : 
{{< codefile file="management/commands/capture_tic.py" lang="python" >}}
from django.core.management.base import BaseCommand
from ticapp.models import Data
from django.utils import timezone
import serial
import time

class Command(BaseCommand):
    help = 'Capture et insère les données TIC en continu depuis le compteur Linky'

    def __init__(self):
        super().__init__()
        # Initialisation du port série
        self.ser = serial.Serial(
            port="/dev/ttyAMA0",
            baudrate=9600,
            bytesize=serial.SEVENBITS,
            parity=serial.PARITY_NONE,
            stopbits=serial.STOPBITS_ONE,
            timeout=1,
            xonxoff=False,
            rtscts=False
        )
        self.listItems = ["DATE", "LTARF", "EAST", "EASF01", "EASF02", "EASD01", "EASD02", "EASD03", "EASD04", "SINSTS", "SINSTS1", "SINSTS2", "SINSTS3"]

    def checkIfDictFull(self, dict, listItems):
        """Vérifie si un dictionnaire contient toutes les clés qui sont dans la liste listItems"""
        setKeys = set(dict.keys())
        return set(listItems).issubset(setKeys)

    def capture_trame(self):
        """Capture une trame TIC complète"""
        while True:
            if self.ser.read(1) == b'\x02':  # Début de trame
                trame = b'\x02'
                while True:
                    byte = self.ser.read(1)
                    trame += byte
                    if byte == b'\x03':  # Fin de trame
                        return trame

    def parse_trame(self, trame):
        """Parse la trame et retourne un dictionnaire avec les données"""
        try:
            data = trame[1:-1].decode('ascii', errors='ignore')
            lines = data.split('\r\n')
            dico = {}
            
            for line in lines:
                if '\t' in line:
                    parts = line.split('\t')
                    key = parts[0]
                    value = parts[1] if len(parts) > 1 else ''
                    dico[key] = value
            
            # Ne retourner que si toutes les clés nécessaires sont présentes
            if self.checkIfDictFull(dico, self.listItems):
                subDico = {key: dico[key] for key in self.listItems if key in dico}
                return subDico
            else:
                return None
                
        except Exception as e:
            self.stderr.write(self.style.ERROR(f'Erreur de parsing: {e}'))
            return None

    def handle(self, *args, **options):
        self.stdout.write(self.style.SUCCESS('=== Démarrage de la capture TIC ==='))
        self.stdout.write(f'Port série: {self.ser.port}')
        self.stdout.write(f'Baudrate: {self.ser.baudrate}')
        
        compteur = 0
        erreurs = 0
        
        try:
            while True:
                # Capture de la trame
                trame = self.capture_trame()
                
                # Parse de la trame
                donnees = self.parse_trame(trame)
                
                if donnees:
                    try:
                        # Insertion dans la base de données
                        Data.objects.create(
                            date=donnees["DATE"][1:],  # Retire le premier caractère
                            larf=donnees["LTARF"],
                            east=donnees["EAST"],
                            easf01=donnees["EASF01"],
                            easf02=donnees["EASF02"],
                            easd01=donnees["EASD01"],
                            easd02=donnees["EASD02"],
                            easd03=donnees["EASD03"],
                            easd04=donnees["EASD04"],
                            sinsts=donnees["SINSTS"],
                            sinsts1=donnees["SINSTS1"],
                            sinsts2=donnees["SINSTS2"],
                            sinsts3=donnees["SINSTS3"]
                        )
                        
                        compteur += 1
                        erreurs = 0  # Reset du compteur d'erreurs
                        
                        # Log périodique
                        if compteur % 100 == 0:
                            self.stdout.write(
                                self.style.SUCCESS(
                                    f'✓ {compteur} enregistrements insérés - '
                                    f'Dernier: {timezone.now().strftime("%H:%M:%S")}'
                                )
                            )
                    
                    except Exception as e:
                        erreurs += 1
                        self.stderr.write(
                            self.style.ERROR(f'Erreur insertion BD: {e}')
                        )
                        if erreurs > 10:
                            self.stderr.write(
                                self.style.ERROR('Trop d\'erreurs consécutives, arrêt')
                            )
                            break
                
                else:
                    # Trame incomplète, on continue sans erreur
                    pass
                
                time.sleep(0.5)  # Petite pause entre les captures
                
        except KeyboardInterrupt:
            self.stdout.write(
                self.style.WARNING(
                    f'\n=== Arrêt demandé par l\'utilisateur ===\n'
                    f'Total enregistrements: {compteur}'
                )
            )
        
        except Exception as e:
            self.stderr.write(self.style.ERROR(f'Erreur fatale: {e}'))
        
        finally:
            # Fermeture propre du port série
            if self.ser.is_open:
                self.ser.close()
                self.stdout.write('Port série fermé')
{{< /codefile >}}
- Créer le service *django-ticserver* : {{< cmd >}}sudo nano /etc/systemd/system/django-ticserver.service{{< /cmd >}}
- Copier/coller le contenu : 
{{< codefile file="/etc/systemd/system/django-ticserver.service" lang="ini" >}}
[Unit]
Description=Django TIC Web Server
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/djangoTIC/ticServer
Environment="PATH=/home/pi/djangoTIC/venv/bin"
ExecStart=/home/pi/djangoTIC/venv/bin/gunicorn --bind 0.0.0.0:8000 --workers 2 ticServer.wsgi:application
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
{{< /codefile >}}
- Créer le service *tic-capture* : {{< cmd >}}sudo nano /etc/systemd/system/django-ticserver.service{{< /cmd >}}
- Copier/coller le contenu : 
{{< codefile file="/etc/systemd/system/tic-capture.service" lang="ini" >}}
[Unit]
Description=Capture des données TIC Linky
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/djangoTIC/ticServer
ExecStart=/home/pi/djangoTIC/venv/bin/python manage.py capture_tic
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
{{< /codefile >}}
- Lancer les services :  
{{< cmd >}}sudo systemctl daemon-reload{{< /cmd >}}  
{{< cmd >}}sudo systemctl enable django-ticserver{{< /cmd >}}  
{{< cmd >}}sudo systemctl start django-ticserver{{< /cmd >}}  
{{< cmd >}}sudo systemctl enable tic-capture{{< /cmd >}}  
{{< cmd >}}sudo systemctl start tic-capture{{< /cmd >}}  
## Mise en production
- Créer une clé de sécurité : {{< cmd >}}python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'{{< /cmd >}}
- Créer un fichier *.env* : {{< cmd >}}nano ~/djangoTIC/ticServer/.env{{< /cmd >}}
- Écrire son contenu en l'adaptant : 
{{< codefile file="djangoTIC/ticServer/.env" lang="bash" >}}
SECRET_KEY=votre-cle-generee-ici
DEBUG=False
ALLOWED_HOSTS=192.168.x.x,localhost
{{< /codefile >}}
- Ouvrir *settings.py* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticServer/settings.py{{< /cmd >}}
- Effacer son contenu et le remplacer par : 
{{< codefile file="ticServer/ticServer/settings.py" lang="python" >}}
"""
Django settings for ticServer project.

Generated by 'django-admin startproject' using Django 5.1.7.

For more information on this file, see
https://docs.djangoproject.com/en/5.1/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/5.1/ref/settings/
"""

from pathlib import Path
import os
from decouple import Config, RepositoryEnv    #pour la production et le masquage des données

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent

# Spécification du chemin exact de .env
config = Config(RepositoryEnv(BASE_DIR / '.env'))

# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/5.1/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = config('SECRET_KEY')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = config('DEBUG', default=False, cast=bool)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=lambda v: [s.strip() for s in v.split(',')])

# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'ticapp.apps.TicappConfig',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware', 
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'ticServer.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'ticServer.wsgi.application'


# Database
# https://docs.djangoproject.com/en/5.1/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
        'TIME_ZONE': 'Europe/Paris',
    }
}


# Password validation
# https://docs.djangoproject.com/en/5.1/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/5.1/topics/i18n/

LANGUAGE_CODE = 'fr'

TIME_ZONE = 'Europe/Paris'

USE_I18N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/5.1/howto/static-files/

STATIC_URL = '/static/'

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

STORAGES = {
    "default": {
        "BACKEND": "django.core.files.storage.FileSystemStorage",
    },
    "staticfiles": {
        "BACKEND": "django.contrib.staticfiles.storage.StaticFilesStorage", 
   },
}

# Default primary key field type
# https://docs.djangoproject.com/en/5.1/ref/settings/#default-auto-field

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
{{< /codefile >}}
- Lancer le *css* (Vérifier de bien être dans *~/djangoTIC/ticServer*) : {{< cmd >}}npx @tailwindcss/cli -i ticapp/static/src/input.css -o ticapp/static/src/output.css --minify{{< /cmd >}}
- Effectuer les migrations :  
{{< cmd >}}python ~/djangotTIC/ticServer/manage.py makemigrations{{< /cmd >}}  
{{< cmd >}}python ~/djangotTIC/ticServer/manage.py migrate{{< /cmd >}} 
- Reconfigurer les fichiers statiques : {{< cmd >}}python ~/djangotTIC/ticServer/manage.py collectstatic --noinput{{< /cmd >}} 
- Rebouter la Raspberry : {{< cmd >}}sudo reboot{{< /cmd >}}
- Sauf erreur ou omission, le site est visible à l'adresse *192.168.x.x:8000*