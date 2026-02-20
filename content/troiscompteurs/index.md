+++
date = '2026-01-07T09:12:36+01:00'
draft = false
h1 = 'Trois Linky, une seule Raspberry Pi'
title = 'Lire plusieurs compteurs Linky sur un seul Raspberry Pi'
description = "Comment connecter et lire simultanément trois compteurs Linky sur un Raspberry Pi : un shield PiTInfo sur le GPIO et deux dongles USB série. Configuration Django pour gérer plusieurs flux TIC."
+++
Un [module PitInfo](https://hallard.me/tag/pitinfo/) ne peut lire les données que d'un seul [compteur Linky](https://fr.wikipedia.org/wiki/Linky), et il n'est possible d'installer qu'un seul *PiTinfo* par [Raspberry pi](https://fr.wikipedia.org/wiki/Raspberry_Pi). Si on veut récupérer les [TéléInformations Client (TIC)](https://www.kelwatt.fr/guide/compteur/electricite/linky-prise-secrete) de plusieurs compteurs, on peut passer par des modules branchés en USB, notamment le [uTeleInfo](https://www.tindie.com/products/hallard/micro-teleinfo-v30/) de [Charles Hallard](https://hallard.me/).

{{< avertissement>}}L'installation de deux *uTeleInfo* n'a pas été réalisée lors de la rédaction de cette page !  
Il est donc possible que certaines commandes ne renvoient pas le résultat escompté ou que les rendus définitifs ne correspondent pas à ce qui était initialement souhaité. {{< /avertissement >}}

## Repérer les modules USB
### Les raisons
- Dans la mesure où on va stocker les informations *TIC* de chaque compteur dans des tables de base de données, il faut identifier de manière unique les flux d'informations dans les différents scripts
- Le PiTinfo ne pose pas de difficultés dans la mesure où il est unique
- Pour les *uTeleInfo* branchés en USB, il est indispensable de bien les distinguer pour qu'au reboutage de la *Raspberry* il n'y ait pas d'inversion des clés USB (deux clés dans le cas présent)
### Récupérer les informations USB
- Brancher le premier module *uTeleInfo*
- Dans le dossier {{< focus >}}/dev{{< /focus >}}, le périphérique devrait apparaître sous la forme {{< focus >}}ttyUSBxx{{< /focus >}}, par exemple *ttyUSB0*
- Vérifier en lançant dans un terminal connecté en *ssh* à la Raspberry : {{< cmd >}}ls -l /dev/ttyUSB*{{< /cmd >}}
- Récupérer les informations USB du *uTeleInfo* : {{< cmd >}}udevadm info -a -n /dev/ttyUSBx{{< /cmd >}}  *(remplacer x par sa valeur numérique)*
- Dans la sortie, repérer des lignes spécifiques du type *(les valeurs sont des exemples)*:  
{{< cmdNocopyGoLine >}}ATTRS{idVendor}=="10c4"  
ATTRS{idProduct}=="ea60"  
ATTRS{serial}=="A50285BI"{{< /cmdNocopyGoLine >}}  
- Noter les valeurs des attributs pertinents
- Faire de même pour le deuxième *uTeleInfo*
### Créer des règles udev
- On crée des [règles udev](https://doc.ubuntu-fr.org/udev) qui vont attribuer à chaque *uTeleInfo* un nom spécifique, ce qui permettra le bon repérage des modules indépendamment de l'ordre au boutage.
- Créer le fichier {{< focus >}}99-teleinfo.rules{{< /focus >}} : {{< cmd>}}sudo nano /etc/udev/rules.d/99-teleinfo.rules{{< /cmd >}}
- Créer deux règles, une par module *uTeleinfo* *(adapter le contenu aux valeurs spécifiques)*:  
{{< codefile file="/etc/udev/rules.d/99-teleinfo.rules" lang="bash">}}
# U-Teleinfo Linky 1
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="A50285BI", SYMLINK+="teleinfo_linky_1"

# U-Teleinfo Linky 2
SUBSYSTEM=="tty", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="ea60", ATTRS{serial}=="A50285CJ", SYMLINK+="teleinfo_linky_2"
{{< /codefile >}}
- Ces règles vont créer deux fichiers dans */dev* :  
{{< cmdNocopyGoLine >}}/dev/teleinfo_linky_1  
/dev/teleinfo_linky_2{{< /cmdNocopyGoLine >}}
- C'est par ces noms *teleinfo_linky_xx* qu'on va appeler les données *TIC* dans les scripts *Django*
### Recharger udev
- On recharge udev avec les commandes :  
{{< cmd >}}sudo udevadm control --reload-rules{{< /cmd>}}  
{{< cmd >}}sudo udevadm trigger{{< /cmd >}}
- *Ou tout simplement en reboutant la Raspberry* : {{< cmd >}}sudo reboot{{< /cmd >}}
### Vérification des règles
- Lancer la commande : {{< cmd >}}ls -l /dev/teleinfo*{{< /cmd>}}
- Le retour est du type :  
{{< cmdNocopyGoLine >}}/dev/teleinfo_linky_1 -> ttyUSB1  
/dev/teleinfo_linky_2 -> ttyUSB0{{< /cmdNocopyGoLine >}}

## La base de données
### Créer les tables de la base de données
- On a donc trois modules susceptibles de capturer les informations de trois compteurs différents : un PitInfo et deux uTeleInfo
- Dans {{< focus >}}models.py{{< /focus >}} on crée trois classes, chacune correspondant à une table
- Si on décide d'appeler ces tables {{< focus >}}Linky0{{< /focus >}} pour le *PiTinfo* et {{< focus >}}Linky1{{< /focus >}}, {{< focus >}}Linky2{{< /focus >}} pour les 2 *uTeleInfo*, le fichier *models.py* ressemblera à :

{{< codefile file="~/djangoTIC/ticServer/models.py" lang="python">}}
from django.db import models

# Create your models here.

class Linky0(models.Model):
    ## Modèle pour le compteur relié au PiTinfo
    dateTime = models.DateTimeField(auto_now_add=True, db_index=True)   # la date et l'heure données par l'ordinateur
    date = models.CharField(max_length=13,blank=True, null=True)        # la date et l'heure données par le TIC
    ltarf = models.CharField(max_length=16,blank=True, null=True)       # Type de tarif en cours
    east = models.IntegerField(blank=True, null=True)                   # Energie active soutirée Fournisseur Consommation totale
    // la suite des étiquettes fournies par ce compteur //

class Linky1(models.Model):
    ## Modèle pour le compteur relié au teleinfo_linky_1
    dateTime = models.DateTimeField(auto_now_add=True, db_index=True)   # la date et l'heure données par l'ordinateur
    date = models.CharField(max_length=13,blank=True, null=True)        # la date et l'heure données par le TIC
    ltarf = models.CharField(max_length=16,blank=True, null=True)       # Type de tarif en cours
    east = models.IntegerField(blank=True, null=True)                   # Energie active soutirée Fournisseur Consommation totale
    // la suite des étiquettes fournies par ce compteur //

class Linky2(models.Model):
    ## Modèle pour le compteur relié au teleinfo_linky_2
    dateTime = models.DateTimeField(auto_now_add=True, db_index=True)   # la date et l'heure données par l'ordinateur
    date = models.CharField(max_length=13,blank=True, null=True)        # la date et l'heure données par le TIC
    ltarf = models.CharField(max_length=16,blank=True, null=True)       # Type de tarif en cours
    east = models.IntegerField(blank=True, null=True)                   # Energie active soutirée Fournisseur Consommation totale
    // la suite des étiquettes fournies par ce compteur //
    
{{< /codefile >}}
- Initialiser les tables **en environnement virtuel** :  
{{< cmd >}}cd ~/djangoTIC/ticServer{{< /cmd >}}  
{{< cmd >}}python manage.py makemigrations{{< /cmd >}}  
{{< cmd >}}python manage.py migrate{{< /cmd >}}

### Créer les commandes pour stocker les données
- Voir le tutoriel [Lancement de la base de données]({{< relref "tuto//part4_lancement-de-la-base-de-donnees.md#remplir-la-base-de-données" >}})
- On crée trois commandes dans {{< focus >}}~/djangoTIC/ticServer/ticapp/management/commands{{< /focus >}}
- La première commande se réfèrera au compteur *Linky0* relié par le *PiTinfo* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/management/commands/capture_tic0.py"{{< /cmd >}}
- Le fichier sera de ce type :
{{< codefile file="management/commands/capture_tic0" lang="python">}}
from django.core.management.base import BaseCommand
from ticapp.models import Linky0     # on importe le modèle du compteur Linky0
from django.utils import timezone
import serial
import time

class Command(BaseCommand):
    help = 'Capture et insère les données TIC en continu depuis le compteur Linky0 : PiTinfo'

    def __init__(self):
        super().__init__()
        # Initialisation du port série
        self.ser = serial.Serial(
            port="/dev/ttyAMA0",            # ici le port du compteur Linky0 : celui du PiTinfo : ttyAMA0
            baudrate=9600,
            bytesize=serial.SEVENBITS,
            ...)

        // Le reste du code : voir le fichier management/commandes/capture_tic.py de la partie 4 du tutoriel

{{< /codefile >}}
- Les deux autres commandes se réfèreront aux compteurs reliés par les *uTeleInfo*
- Les fichiers seront de ce type :
{{< codefile file="management/comands/capture_tic1" lang="python">}}
from django.core.management.base import BaseCommand
from ticapp.models import Linky1     # on importe le modèle du compteur Linky1
from django.utils import timezone
import serial
import time

class Command(BaseCommand):
    help = 'Capture et insère les données TIC en continu depuis le compteur Linky1 : uTeleInfo teleinfo_linky_1'

    def __init__(self):
        super().__init__()
        # Initialisation du port série
        self.ser = serial.Serial(
            port="/dev/teleinfo_linky_1",            # ici le port du compteur Linky1 : celui du uTeleInfo: teleinfo_linky_1
            baudrate=9600,
            bytesize=serial.SEVENBITS,
            ...)

        // Le reste du code : voir le fichier management/commandes/capture_tic.py de la partie 4 du tutoriel
{{< /codefile >}}
### Créer les services des commandes
- Voir le tutoriel [Lancement de la base de données]({{< relref "tuto//part4_lancement-de-la-base-de-donnees.md#lancer-le-script-de-captage-des-données" >}})
- Créer le premier service : {{< cmd >}}sudo nano /etc/systemd/system/tic-capture0.service{{< /cmd >}}
- Le code est le suivant : 
{{< codefile file="/etc/systemd/system/tic-capture0.service" lang="ini" >}}
[Unit]
# description pour le Linky0 qui est sur le PiTinfo
Description=Capture des données TIC Linky0 (PiTinfo)  
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/djangoTIC/ticServer
# commande capture_tic0 du Linky0
ExecStart=/home/pi/djangoTIC/venv/bin/python manage.py capture_tic0      
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
{{< /codefile >}}
- On crée les deux autres services en procédant de même : modifier les valeurs des commandes
- On lance les services et on les rend pérennes :  
{{< cmd >}}sudo systemctl daemon-reload{{< /cmd >}}  
{{< cmd >}}sudo systemctl enable tic-capture0{{< /cmd >}}  
{{< cmd >}}sudo systemctl enable tic-capture1{{< /cmd >}}  
{{< cmd >}}sudo systemctl enable tic-capture2{{< /cmd >}}  
{{< cmd >}}sudo systemctl start tic-capture0{{< /cmd >}}  
{{< cmd >}}sudo systemctl start tic-capture1{{< /cmd >}}  
{{< cmd >}}sudo systemctl start tic-capture2{{< /cmd >}}  
## Le fichier views.py
On créé les différentes fonctions qui vont permettre de lire les données récupérées dans la base de données
### Fonction *index*
- Dans la page d'accueil on veut, par exemple, afficher les intensités instantanées de chacun des compteurs
- Il faut appeler les trois modèles dans *views.py* : {{< cmd >}}from .models import Linky0, Linky1, Linky2{{< /cmd >}}
- Renvoyer le *context* pour l'affichage dans le gabarit *index.html* : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/views.py{{< /cmd >}}
- Le fichier *views.py* sera de ce type : 
{{< codefile file="ticServer/ticapp/views.py" lang="python" >}}
from django.shortcuts import render
from django.utils import timezone           
from .models import Linky0, Linky1, Linky3           # new

def index(request):                         
    now = timezone.now()
    data0 = Linky0.objects.last()           # dernier enregistrement dans la table Linky0
    data1 = Linky1.objects.last()           # dernier enregistrement dans la table Linky1
    data2 = Linky2.objects.last()           # dernier enregistrement dans la table Linky2
    
    # les trois valeurs à récupérer
    instant0 = format(data0.sinsts, ',').replace(',', ' ')
    instant1 = format(data1.sinsts, ',').replace(',', ' ')
    instant2 = format(data2.sinsts, ',').replace(',', ' ')
    
    # création du dictionnaire
    values = {
        'instant0': instant0, 
        'instant1': instant1, 
        'instant2': instant2, 
    }

    # création du contexte
    context = {
        'values': values,
        'now': now,
    }

    return render(request, 'index.html', context)
{{< /codefile >}}
### Autres fonctions
Les autres fonctions peuvent être modifiées de même pour permettre l'affichage soit d'un seul compteur, soit de tous les compteurs.
## Les gabarits html
### index.html
- Maintenant que la fonction *index* est déterminée, il faut adapter le fichier *index.html* pour afficher les données : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/templates/index.html{{< /cmd >}}
- Le code sera de ce type :
{{< codefile file="ticapp/templates/index.html" lang="html" >}}
{% extends "_base.html" %}
{% load l10n %}
{% load tz %}

{% block content %}
<div class="max-w-3xl mx-auto text-center mt-12">
    <h1 class="text-xl md:text-3xl lg:text-4xl font-bold text-gray-700 leading-tight mb-2 border-b-2 border-gray-400 pb-2">
      Puissances Instantanées (en Watt)
    </h1>
</div>

{% timezone "Europe/Paris" %}

<div class="flex items-center justify-center pt-2 md:pt-4">
    <p class="font-semibold text-gray-500">{{ now | date:" D j F Y - H:i:s "}}</p>   
    <p class="text-gray-400 pl-4"></p>
</div>
{% endtimezone %}

<div class="flex items-center justify-center pt-2 md:pt-4">
    <table class="border-separate border-spacing-x-8 md:border-spacing-x-16 border-spacing-y-4 md:border-spacing-y-8 border border-gray-200">
        <tbody>
            <tr class="">
                <td class="font-semibold ">Compteur Linky 0</td>         
                <td class="font-semibold text-right ">{{  values.instant0 }}</td>
            </tr>
            <tr>
                <td class="">Compteur Linky 1</td>
                <td class="text-right">{{  values.instant1 }}</td>
            </tr>
            <tr>
                <td class="">Compteur Linky 2</td>  
                <td class="text-right">{{  values.instant2 }}</td>
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
### Les autres gabarits
En fonction des besoins d'affichage, on modifie (ou on crée) les gabarits nécessaires.