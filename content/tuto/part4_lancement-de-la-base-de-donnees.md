+++
date = '2025-12-17T07:09:22+01:00'
draft = false
title = 'Partie 4 : Lancement de la Base de Données'
description = "Sur Django, activation de la base de données avec utilisation de l’ORM pour stocker les valeurs. Un service systemd lance automatiquement le script de captage"
+++

{{< line >}}
## Quelles données stocker
Les données qu'on souhaite stocker dans la base de données sont les suivantes :  
- DATE  
- LTARF 
- EAST, EASF01, EASF02  
- EASD01, EASD02, EASD03, EASD04 
- SINSTS, SINSTS1, SINSTS2, SNSIST3  
Au besoin, en fonction des informations désirées, il est possible de recueillir d'autres champs.
{{< line >}}
## Fichier models.py
### Création du fichier models.py
- Ces données doivent être déclarées dans un fichier {{< focus >}}models.py{{< /focus >}}
- Ce fichier existe déjà dans le dossier {{< focus >}}ticapp{{< /focus >}}
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/models.py{{< /cmd >}}
- Copier/coller le contenu suivant : 
{{< codefile file="ticServer/ticapp/models.py" lang="python" >}}
from django.db import models

# Create your models here.

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
    sinsts3 = models.IntegerField(blank=True, null=True)                # Puissance apparente instantanée soutirée phase 3
{{< /codefile >}}
- Sauvegarder et quitter

### Commentaire sur les données
- dateTime est une date, caractérisée par le champ DateTimeField ; il s'agit de la date au moment de la création des données
- date est un *string*, qui comprend 13 caractères ; c'est une date du type H251228062542 pour le 28 décembre 2025 à 6H25mn42s
- ltarf est un *string* de longueur maximale 16 caractères
- les autres champs sont des données numériques entières
### Créer la table dans la base de données
- Le fichier {{< focus >}}models.py{{< /focus >}} ayant été modifié, il faut faire prendre en compte ces modifications par Django
- Se placer dans le dossier qui contient {{< focus >}}manage.py{{< /focus >}} : {{< cmd >}}cd ~/djangoTIC/ticServer{{< /cmd >}}
- Lancer les deux commandes suivantes :  
{{< cmd >}}
python manage.py makemigrations
{{< /cmd >}}  
{{< cmd >}}
python manage.py migrate
{{< /cmd >}}
- Le modèle s'appelant ***Data*** et l'application ***ticapp***, ces commandes vont créer la table ***ticapp_data*** dans la base de données

{{< line >}}
## Remplir la base de données
### Introduction
- Une fois créée la table {{< focus >}}ticapp_data{{< /focus >}} avec ses colonnes bien définies, il faut lui fournir les données
- Pour cela il faut adapter le fichier de capture de données {{< focus >}}~/djangoTIC/scripts/readTIC_test.py{{< /focus >}} de manière à ce que les données soient versées dans la base de données.  
- Plutôt que d'utiliser du [langage SQL](https://fr.wikipedia.org/wiki/Structured_Query_Language) brut pour rentrer ces données, il est préférable de laisser Django manipuler celles-ci en utilisant son [ORM](https://fr.wikipedia.org/wiki/Mapping_objet-relationnel), qui va directement faire le lien entre notre code python et la base de données
- Ceci est d'autant plus nécessaire que la gestion des dates dans la base de données peut poser des problèmes de [dates *naïves* et *avisées*](https://docs.python.org/fr/3.5/library/datetime.html) (naive/aware) avec risque d'erreur dans les calculs de temps écoulé entre deux dates
- L'une des manières adaptées pour fournir la base de données est de passer par une [commande personnalisée](https://www.formation-django.fr/framework-django/zoom-sur/commandes-management.html)
### Création de la commande *capture_tic*
- Créer les dossiers nécessaires : {{< cmd >}}mkdir -p ~/djangoTIC/ticServer/ticapp/management/commands{{< /cmd >}}
- Transformer ces dossiers en modules python en y créant un fichier {{< focus >}}__init__.py{{< /focus >}} :  
{{< cmd >}}touch ~/djangoTIC/ticServer/ticapp/management/\_\_init\_\_.py{{< /cmd >}}  
{{< cmd >}}touch ~/djangoTIC/ticServer/ticapp/management/commands/\_\_init\_\_.py{{< /cmd >}}
- Créer et ouvrir le fichier {{< focus >}}capture_tic{{< /focus >}} : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/management/commands/capture_tic.py{{< /cmd >}}
- Copier le code suivant *(adapté du script readTIC_test.py)* :  
{{< codefile file="management/commandes/capture_tic.py" lang="python" >}}
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
            
            # Ne renvoyer que si toutes les clés nécessaires sont présentes
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
- Sauvegarder et quitter : {{< focus >}}Ctrl-O{{< /focus >}} {{< focus >}}Entrée{{< /focus >}} {{< focus >}}Ctrl-X{{< /focus >}}
### Lancer le script de captage des données
- Se mettre, en **environnement virtuel**, dans {{< focus >}}ticServer{{< /focus >}} : {{< cmd >}}cd ~/djangoTIC/ticServer{{< /cmd >}}
- Lancer la commande : {{< cmd >}}python manage.py capture_tic{{< /cmd >}}
- La base de données commence à stocker les données du Linky
### Rendre le captage et le stockage des données pérennes
- Le captage et le stockage doivnet se faire automatiquement au reboutage de la Raspberry
- Pour cela on crée un [service systemd](https://blog.stephane-robert.info/docs/admin-serveurs/linux/services/)
- Créer le service : {{< cmd >}}sudo nano /etc/systemd/system/tic-capture.service{{< /cmd >}}
- Coller le code suivant :
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
- Sauvegarder et quitter
- Charger le service :  
{{< cmd >}}sudo systemctl daemon-reload{{< /cmd >}}  
{{< cmd >}}sudo systemctl enable tic-capture{{< /cmd >}}  
{{< cmd >}}sudo systemctl start tic-capture{{< /cmd >}}
- Tester le service : {{< cmd >}}sudo systemctl status tic-capture{{< /cmd >}}
- Le système démarrera automatiquement à chaque boutage de la Raspberry