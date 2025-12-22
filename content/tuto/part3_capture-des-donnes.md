+++
date = '2025-12-16T22:10:22+01:00'
draft = false
title = 'Partie 3 : Capture des Données'
description = "Structure des données fournies par la TIC d’un Linky ; script de capture des données en ne conservant que les étiquettes souhaitées (triphasé avec mode jour/nuit)"
+++
{{< line >}}
## Structure des données
La TéléInformation Client envoie en continu les données du compteur Linky. Ces données sont structurées sous forme de trames. Chaque trame est délimitée par un caractère de début {{< focus >}}STX (b'\x02'){{< /focus >}} et un caractère de fin {{< focus >}}ETX (b'\x03'){{< /focus >}}.
Au sein de chaque trame, des étiquettes vont s'afficher avec, en correspondance, les valeurs attribuées à ces étiquettes. La liste des étiquettes varie selon que l'on est en mode *historique* ou *standard*, selon que l'on est en monophasé ou en triphasé, selon que l'on bénéficie ou non d'un tarif heures pleines / heures creuses.
La liste des étiquettes est disponible chez [Enedis](https://www.enedis.fr/media/2035/download).  

Les étiquettes qui vont nous intéresser sont :  
- DATE qui affiche la date et l'heure courante saisies par le Linky  
- LTARF qui précise si on est en heures creuses ou en heures pleines  
- EAST qui est l'énergie soutirée totale depuis la remise à zéro du Linky, en Wh  
- EASF01 qui correspond aux heures creuses en Wh  
- EASF02 qui correspond aux heures pleines en Wh  
- EASD01, EASD02, EASD03, EASD04 qui sont les énregies actives soutirées par le Distributeur  
- SINSTS, SINSTS1, SINSTS2 et SNSIST3 qui sont les puissances apparentes soutirées en totalité, pour la phase1, pour la phase2, pour la phase3  
{{< line >}}
## Installer pyserial
- [pyserial](https://pyserial.readthedocs.io/en/latest/pyserial.html) est un module python qui permet de récupérer les données émises par un optocoupleur
- Se mettre en environnement virtuel
- Installer {{< focus >}}pyserial{{< /focus >}} en tapant : {{< cmd >}}pip install  pyserial{{< /cmd >}}
{{< line >}}
## Script de capture des données TIC
### Script de test
- *Ce script de test n'est pas utile pour le projet en soi*
- *Il a un but didactique : comprendre le code de capture des données du TIC*
- Créer un dossier {{< focus >}}script{{< /focus >}} : {{< cmd >}}mkdir ~/djangoTIC/scripts{{< /cmd >}}
- Ouvrir en écriture le script {{< focus >}}readTIC_test.py{{< /focus >}} : {{< cmd >}}`nano ~/djangoTIC/scripts/readTIC_test.py{{< /cmd >}}
- Insérer le code suivant : 


{{< codefile file="djangoTIC/scripts/readTIC_test.py" lang="python" >}}
import serial
import time

# Initialisation du port série (le port s'ouvre automatiquement) avec les paramètres ad hoc
ser = serial.Serial(
    port="/dev/ttyAMA0",            
    baudrate=9600,
    bytesize=serial.SEVENBITS,
    parity=serial.PARITY_NONE,
    stopbits=serial.STOPBITS_ONE,
    timeout=1
)

# la liste des étiquettes dont on souhaite récupérer les valeurs
listItems = ["DATE", "LTARF", "EAST", "EASF01", "EASF02", "EASD01", "EASD02", "EASD03", "EASD04","SINSTS", "SINSTS1", "SINSTS2", "SINSTS3"]

def checkIfDictFull(dict, listItems):
    """Vérifie si un dictionnaire contient toutes les clés qui sont dans la liste listItems
    Lié au fait que le flux du TIC ne vas pas nécessairement fournir toutes les données pour un même jeu de données"""
    setKeys = set(dict.keys())
    if set(listItems).issubset(setKeys):
        return True
    else:
        return False


def capture_trame():
    while True:
        if ser.read(1) == b'\x02':      # Début de trame
            trame = b'\x02'             # On crée une chaîne de caractères
            while True:
                byte = ser.read(1)      # On lit le caractère
                trame += byte           # On ajoute le caractère à la chaîne
                if byte == b'\x03':     # Fin de trame
                    return trame        

def parse_trame(trame):
    data = trame[1:-1].decode('ascii', errors='ignore')                 # décodage du binaire en ascii
    print(f'type de data = {type(data)}')
    lines = data.split('\r\n')                                          # transformer la chaîne de caractères en une succession de lignes
    print(f'type de lines = {type(lines)}')
    dico = {}
    for line in lines:                                                  # traiter les lignes une par une ; les éléments de lignes sont séparés par des tabulations
        key = line.split('\t')[0]                                       # capter l'étiquette de la ligne
        value = line.split('\t')[1] if '\t' in line else ''             # capter la valeur de l'étiquette
        dico[key] = value                                               # ajouter l'étiquette et sa valeur dans le dictionnaire dico
        # print(f'Clé: {key}, Valeur: {value}')
    subDico = {key: dico[key] for key in listItems if key in dico}      # sélectionner les étiquettes et les valeurs qui sont dans la liste listitems
        
    if checkIfDictFull(dico, listItems):
        print('Dictionnaire complet capturé')
        # return subDico
    else:
        print('Dictionnaire incomplet')

try:
    while True:                                                         # boucle d'impression des données
        trame = capture_trame()
        donnees = parse_trame(trame)
        print(donnees)
        time.sleep(1)                                                   # Pause entre les captures
except KeyboardInterrupt:
    print("\nArrêt du script par l'utilisateur.")                       # Ctrl+C pour interrompre le script
except Exception as e:
    print(f"Erreur : {e}")
{{< /codefile >}}

### Lancer le script de test
- En environnement virtuel
- Lancer la commande : {{< cmd >}}python ~/djangoTIC/scripts/readTIC_test.py{{< /cmd >}}
- Les trames défilent, en indiquant soit que le dictionnaire est incomplet, soit qu'il est complet.
- Interrompre avec {{< focus >}}Ctrl-C{{< /focus >}}
### Conserver le script ?
- Une fois testé, le dossier et le script peuvent être gardés ou supprimés
- Pour les supprimer : {{< cmd >}}rm -R ~/djangoTIC/scripts{{< /cmd >}}
