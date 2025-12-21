+++
date = '2025-12-15T22:58:48+01:00'
draft = false
title = 'Partie 2 : Préparation des Logiciels'
+++
{{< line >}}

## Vérifier le fonctionnement du module PiTinfo
### Se connecter en ssh à la Raspberry
- Sur son ordinateur, ouvrir un terminal de commande (la méthode est différente selon l'OS utilisé : [consulter cette page](https://djangocamera.netlify.app/tuto/part1/#connexion-ssh-%C3%A0-la-raspberry))
- Taper : {{< cmd >}}ssh pi@<local IP Raspberry>{{< /cmd >}}
- Admettons que l'IP locale de la Raspberry soit {{< focus >}}192.168.1.120{{< /focus >}}, taper : {{< focus >}}ssh pi@192.168.1.120{{< /focus >}} (à condition, bien sûr, d'avoir laissé {{< focus >}}pi{{< /focus >}} comme nom d'utilisateur de la Raspberry)
- Entrer le mot de passe de la Raspberry
- Si besoin, autoriser la connexion en acceptant la création d'une clé
### Installer picocom
- [picocom](https://micropython.fr/05.outils/terminal_serie/picocom/) est un émulateur de terminal qui va permettre de communiquer avec le PiTinfo
- Installer {{< focus >}}picocom{{< /focus >}} en tapant dans le terminal connecté à la Raspberry en ssh : {{< cmd >}}sudo install picocom {{< /cmd >}}
### Modifier le fichier /boot/firmware/config.txt
- Pour assurer un bon fonctionnement de la commande {{< focus >}}picocom{{< /focus >}} il est nécessaire de modifier le fichier {{< focus >}}/boot/firmware/config.txt{{< /focus >}}
- Taper : {{< cmd >}}sudo nano /boot/firmware/config.txt{{< /cmd >}} (obligé d'être en superutilisateur, car le fichier est protégé)
- Au paragraphe {{< focus >}}All{{< /focus >}} vérifier la valeur de {{< focus >}}enable_uart{{< /focus >}}
- Elle doit être égale à 1 ; si elle est à 0, la modifier
- Ajouter une ligne supplémentaire (pour désactiver le bluetooth) : {{< cmd >}}dtoverlay=disable-bt{{< /cmd >}}
- *Si le bluetooth n'est pas désactivé, la lecture des données pourra être mauvaise*
- Sauvegarder et quitter : {{< focus >}}Ctrl-O{{< /focus >}}{{< focus >}}Entrée{{< /focus >}}{{< focus >}} Ctrl-X{{< /focus >}}
### Tester la commande 
- Dans le terminal lancer la commande de lecture des données : {{< cmd >}}picocom -b 9600 -d 7 -p e -f h /dev/ttyAMA0{{< /cmd >}}
- Les données de la TIC défilent en continu dans le terminal
- Pour interrompre, taper : {{< focus >}}Ctrl-A{{< /focus >}} {{< focus >}}Ctrl-Q{{< /focus >}}
- La signification des drapeaux de la commande sont disponibles sur la [page man](https://manpages.ubuntu.com/manpages/bionic/man1/picocom.1.html)
- Il convient d'être en 9600 bauds (et non 1200), d'avoir une parité paire ({{< focus >}}-p e{{< /focus >}}) et d'être à 7 bits par caractère ({{< focus >}}-d 7{{< /focus >}}), ainsi que le préconise Enedis dans sa [notice de téléinformation](https://www.enedis.fr/media/2035/download)
{{< line >}}
## Installer Django
Pour assurer une visualisation agréable des données, on va ouvrir des pages contenant les informations souhaitées dans un navigateur web. La méthode utilisée, ici, est de passer par un site internet créé avec [Django](https://www.djangoproject.com/).
### Création d'un environnement virtuel
#### *python3-venv* est-il installé ?
Pour fonctionner efficacement sur une Raspberry, il est souhaitable de se mettre en environnement virtuel. Cet environnement évite les conflits, toujours possibles, entre différentes versions de [Python](https://fr.wikipedia.org/wiki/Python_(langage)). *A priori* la commande pour lancer la création d'un environnement virtuel est installée par défaut.
- Vérifier si {{< focus >}}python3-venv{{< /focus >}}est installé en tapant dans le terminal : {{< cmd >}}dpkg -l | grep venv{{< /cmd >}}
- Si vous avez une réponse du type :

{{< cmdNocopy >}}
ii  python3-venv                                     3.11.2-1+b1                               arm64        venv module for python3 (default python3 version)
ii  python3.11-venv                                  3.11.2-6+deb12u6                          arm64        Interactive high-level object-oriented language (pyvenv binary, version 3.11)
{{< /cmdNocopy >}}
cela signifie que {{< focus >}}python3-venv{{< /focus >}}est installé
- Dans la négative, l'installer en tapant : {{< cmd >}}sudo apt install python3-venv{{< /cmd >}}  *(au besoin après avoir mis à jour les différents paquets avec {{< cmd >}}sudo apt update{{< /cmd >}})*
#### Créer le dossier de travail
- Il convient, en premier lieu, de créer le dossier dans lequel sera hébergé le site
- On va appeler, par exemple, ce dossier {{< focus >}}djangoTIC{{< /focus >}}
- Dans le terminal, taper : {{< cmd >}}mkdir djangoTIC{{< /cmd >}}
- On vérifie sa création : {{< cmd >}}ls{{< /cmd >}}
- Au passage, on peut supprimer les différents dossiers inutiles préexistants, par exemple {{< focus >}}Desktop{{< /focus >}} en tapant : {{< cmd >}}rm -R Desktop{{< /cmd >}}
#### Créer *venv*
- Aller dans le dossier {{< focus >}}djangoTIC{{< /focus >}}: {{< cmd >}}cd djangoTIC{{< /cmd >}}
- Créer l'environnement virtuel que l'on va appeler {{< focus >}}venv{{< /focus >}}: {{< cmd >}}python3 -m venv venv{{< /cmd >}}
- On vérifie que l'environnement virtuel est créé avec la commande {{< focus >}}ls{{< /focus >}}
- Pour activer l'environnement virtuel on tape la commande : {{< cmd >}}source ~/djangoTIC/venv/bin/activate{{< /cmd >}}
- L'invite de commande montre que l'on est en environnement virtuel : {{< focus >}}(venv) pi@raspberrypi:~ ${{< /focus >}}
- Pour désactiver l'environnement virtuel : {{< cmd >}}deactivate{{< /cmd >}}
### Installer Django
- Se mettre en environnement virtuel et se placer dans le dossier {{< focus >}}djangoTIC{{< /focus >}} :  
{{< cmd >}}
source ~/djangoTIC/venv/bin/activate
{{< /cmd >}}  
{{< cmd >}}
cd ~/djangoTIC
{{< /cmd >}}
- Lancer la commande d'installation de {{< focus >}}Django{{< /focus >}}: {{< cmd >}}python -m pip install Django{{< /cmd >}}
- *Comme on est en environnement virtuel, inutile de préciser {{< focus >}}python3{{< /focus >}} : il n'y a pas d'ambiguité*
- Vérifier l'installation : {{< cmd >}}python -m django --version{{< /cmd >}} qui doit renvoyer une sortie du type : {{< focus >}}5.2.9{{< /focus >}} en fonction du nom de la version
### Créer le projet Django
- On va appeler ce projet {{< focus >}}ticServer{{< /focus >}}
- Vérifier qu'on est en environnement virtuel dans le dossier {{< focus >}}djangoTIC{{< /focus >}}
- Lancer la commande {{< cmd >}}django-admin startproject ticServer{{< /cmd >}}
- Django a créé un dossier {{< focus >}}ticServer{{< /focus >}}
- L'arborescence des dossiers et fichiers est du type : 

{{< tree >}}
/home/pi/djangoTIC
├── ticServer
│   ├── manage.py
│   └── ticServer
│       ├── asgi.py
│       ├── __init__.py
│       ├── settings.py
│       ├── urls.py
│       └── wsgi.py
└── venv 
{{< /tree >}}

### Créer l'application *ticapp*
- Dans le projet {{< focus >}}ticServer{{< /focus >}}on crée une application que l'on va appeler {{< focus >}}ticapp{{< /focus >}}
- Si on décide de changer le nom de cette application, **ne pas mettre de majuscule** dans ce nom, pour éviter des conflits avec les paramétrages qui vont être créés par Django
- Vérifier qu'on est bien dans le dossier qui contient le fichier {{< focus >}}manage.py{{< /focus >}} et, si besoin, s'y placer : {{< cmd >}}cd ~/djangoTIC/ticServer{{< /cmd >}}
- Lancer la commande de création de l'application : {{< cmd >}}python manage.py startapp ticapp{{< /cmd >}}
- Django a créé un dossier {{< focus >}}ticapp{{< /focus >}} dans le dossier {{< focus >}}djangoTIC/ticServer{{< /focus >}}
- L'arborescence est maintenant :
{{< tree >}}
/home/pi/djangoTIC
├── ticServer
│   ├── manage.py
│   ├── ticapp
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   ├── models.py
│   │   ├── tests.py
│   │   └── views.py
│   └── ticServer
│       ├── asgi.py
│       ├── __init__.py
│       ├── __pycache__
│       ├── settings.py
│       ├── urls.py
│       └── wsgi.py
└── venv
{{< /tree >}}


### Modifier les paramètres dans settings.py
- Ouvrir en écriture le fichier {{< focus >}}settings.py{{< /focus >}} : {{< cmd >}}nano ~/djangoTIC/ticServer/ticServer/settings.py{{< /cmd >}}
- Modifier {{< focus >}}ALLOWED_HOSTS{{< /focus >}} en ajoutant l'IP de la raspberry ; par exemple{{< focus >}}
ALLOWED_HOSTS = ["192.168.1.120"]{{< /focus >}} *(à modifier en fonction de l'IP attribuée à la Raspberry)*
- Ajouter l'application {{< focus >}}ticapp{{< /focus >}} dans {{< focus >}}INSTALLED_APPS{{< /focus >}} en insérant la valeur : {{< focus >}}'ticapp.apps.TicappConfig',{{< /focus >}} 
la liste devient : 
{{< codefile file="ticServer/ticServer/settings.py" lang="python">}}
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'ticapp.apps.TicappConfig',        # new     
]
{{< /codefile >}}
- Ajouter {{< focus >}}TIME_ZONE : 'Europe/Paris'{{< /focus >}} dans {{< focus >}}DATABASES{{< /focus >}} *(il n'est pas utile de modifier le nom de la base de données)*
- Le dictionnaire {{< focus >}}DATABASES{{< /focus >}} devient :
{{< codefile file="ticServer/ticServer/settings.py" lang="python" >}}
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
        'TIME_ZONE': 'Europe/Paris',
    }
}
{{< /codefile >}}
- Modifier la valeur de {{< focus >}}LANGUAGE_CODE{{< /focus >}} : {{< focus >}}LANGUAGE_CODE = 'fr'{{< /focus >}}
- Modifier la valeur de {{< focus >}}TIME_ZONE{{< /focus >}} : {{< focus >}}TIME_ZONE = 'Europe/Paris'{{< /focus >}}
- Sauvegarder le fichier et quitter  : {{< focus >}}Ctrl-O Entrée Ctrl-X{{< /focus >}}

### Vérifier dans un navigateur
- Vérifier que l'environnement virtuel est lancé
- Se placer dans le dossier : {{< cmd >}}cd ~/djangoTIC/djangoServer{{< /cmd >}}
- Lancer la commande : {{< cmd >}}python manage.py runserver 0.0.0.0:8000{{< /cmd >}}
- *Ne pas tenir compte des avertissements qui sont normaux à ce stade*
- Dans un navigateur de votre ordinateur, taper l'adresse : {{< focus >}}192.168.1.120:8000{{< /focus >}} *(à modifier en fonction de l'adresse IP locale)*
- Vous devez voir la page d'accueil de Django
<div class="w-2/3 md:w-1/3 pt-4 pl-2">
{{< imgResize src=accueilDjango.jpg alt="page d'accueil de django indiquant que l'installation est réussie" >}}
</div>
<br>

- **Dans la commande la valeur 0.0.0.0 permet d'accéder au site depuis un autre appareil que la Raspberry**
- 8000 représente le port classiquement utilisé par Django
{{< line >}}
## Installer Tailwind et Flowbite
Pour personnaliser l'aspect du site qui va afficher les données, on doit utiliser un [fichier css](https://fr.wikipedia.org/wiki/Feuilles_de_style_en_cascade).  
Plutôt que d'écrire soi-même le {{< focus >}}css{{< /focus >}}, le plus simple est de passer par un outil de développement dédié. Le choix porte sur [tailwind](https://tailwindcss.com/) et sa bibliothèque intégrée [flowbite](https://flowbite.com/).
### Installer nodejs et npm
- Pour installer {{< focus >}}tailwind{{< /focus >}}et {{< focus >}}flowbite{{< /focus >}} on utilise la commande {{< focus >}}npm{{< /focus >}}
- On installe tout d'abord {{< focus >}}nodejs{{< /focus >}}: {{< cmd >}}sudo apt install nodejs{{< /cmd >}}
- On vérifie l'installation : {{< cmd >}}node -v{{< /cmd >}} qui renvoie le numéro de version
- On installe {{< focus >}}npm{{< /focus >}} : {{< cmd >}}sudo apt install npm{{< /cmd >}}
- On vérifie l'installation : {{< cmd >}}npm -v{{< /cmd >}} qui renvoie le numéro de version
### Installer tailwind
- Suivre la procédure décrite dans [Tailwind CLI](https://tailwindcss.com/docs/installation/tailwind-cli)
- Se mettre **en environnement virtuel** dans le dossier {{< focus >}}~/djangoTIC/ticServer{{< /focus >}} : {{< cmd >}}cd ~/djangoTIC/ticServer{{< /cmd >}}
- Taper la commande : {{< cmd >}}npm install tailwindcss @tailwindcss/cli{{< /cmd >}}
- Dans le dossier {{< focus >}}~/djangoTIC/ticServer{{< /focus >}} ont été créés : le dossier {{< focus >}}node_modules{{< /focus >}} et les deux fichiers {{< focus >}}package-lock.json{{< /focus >}}et {{< focus >}}package.json{{< /focus >}}
- Créer les dossiers {{< focus >}}static{{< /focus >}}et {{< focus >}}src{{< /focus >}} en lançant la commande : {{< cmd >}}mkdir -p ~/djangoTIC/ticServer/ticapp/static/src{{< /cmd >}}
- Créer et ouvrir le fichier {{< focus >}}input.css{{< /focus >}} en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/static/src/input.css{{< /cmd >}}
- Dans le fichier écrire : {{< cmd >}}@import "tailwindcss";{{< /cmd >}}
- Sauvegarder et fermer
- En étant toujours dans le dossier {{< focus >}}~/djangoTIC/ticServer{{< /focus >}} lancer l'[interface de ligne de commande](https://fr.wikipedia.org/wiki/Interface_en_ligne_de_commande) en tapant la commande : {{< cmd >}}npx @tailwindcss/cli -i ticapp/static/src/input.css -o ticapp/static/src/output.css --watch{{< /cmd >}}
- Le fichier {{< focus >}}output.css{{< /focus >}} a été créé ; c'est ce fichier qui va contenir le css du site

### Installer flowbite
- Suivre la procédure décrite dans [Flowbite Quickstart](https://flowbite.com/docs/getting-started/quickstart/)
- Se mettre **en environnement virtuel** dans le dossier {{< focus >}}~/djangoTIC/ticServer{{< /focus >}}
- Taper la commande : {{< cmd >}}npm install flowbite{{< /cmd >}}
- Dans le dossier {{< focus >}}node_modules{{< /focus >}} un dossier {{< focus >}}flowbite{{< /focus >}} (entre autres) a été créé
- Pour des questions de chemin des fichiers statiques, il peut être préférable de copier le dosser {{< focus >}}flowbite{{< /focus >}} de {{< focus >}}nodes_modules{{< /focus >}} vers {{< focus >}}static{{< /focus >}} : {{< cmd >}}cp -r ~djangoTIC/ticServer/node_modules/flowbite ~djangoTIC/ticServer/ticapp/static/{{< /cmd >}}
- Modifier le fichier {{< focus >}}input.css{{< /focus >}} en l'ouvrant en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticapp/static/src/input.css{{< /cmd >}}
- Ajouter les lignes : 

{{< codefile file="static/src/input.css" lang="css" >}}
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
@import "flowbite/src/themes/default";
@plugin "flowbite/plugin";
@source "../flowbite";
{{< /codefile >}}


- Le fichier final ressemble à : 
{{< codefile file="static/src/input.css" lang="css" >}}
@import "tailwindcss";
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
@import "flowbite/src/themes/default";
@plugin "flowbite/plugin";
@source "../flowbite";
{{< /codefile >}}



### Simplification de la commande *npx*
Pour modifier en continu le fichier {{< focus >}}output.css{{< /focus >}}, il faut donc lancer dans un terminal la commande {{< focus >}}npx etc...{{< /focus >}}.  
Pour que la commande soit prise en compte il faut :
- être en environnement virtuel ou pas !
- être dans le bon dossier : {{< focus >}}~/djangotTIC/ticServer{{< /focus >}} car c'est dans ce dossier qu'est placé le fichier {{< focus >}}package.json{{< /focus >}}
**On peut tout à fait se contenter de suivre cette procédure.**
Il est cependant possible de simplifier le lancement de la commande en modifiant le fichier {{< focus >}}package.json{{< /focus >}} :
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/package.json{{< /cmd >}}
- Ajouter les lignes (on est dans un [json](https://fr.wikipedia.org/wiki/JavaScript_Object_Notation), donc attention aux virgules) : 
{{< codefile file="ticServer/package.json" lang="json" >}}
,
"scripts": {
    "dev": "npx @tailwindcss/cli -i ticapp/static/src/input.css -o ticapp/static/src/output.css --watch",
    "build": "npx @tailwindcss/cli -i ticapp/static/src/input.css -o ticapp/static/src/output.css --minify"
    }
{{< /codefile >}}
- Le fichier modifié est le suivant (à adapter aux versions des logiciels): 
{{< codefile file="ticServer/package.json" lang="json" >}}
{
  "dependencies": {
    "@tailwindcss/cli": "^4.1.10",
    "flowbite": "^3.1.2"
  },
  "scripts": {
    "dev": "npx @tailwindcss/cli -i ticapp/static/src/input.css -o ticapp/static/src/output.css --watch",
    "build": "npx @tailwindcss/cli -i ticapp/static/src/input.css -o ticapp/static/src/output.css --minify"
  }
}
{{< /codefile >}}
Avec cette modification, la commande à lancer dans {{< focus >}}djangoTIC/ticServer{{< /focus >}} est la suivante : {{< cmd >}}npm run dev{{< /cmd >}}  
Si on veut éviter d'être nécessairement dans le dossier {{< focus >}}djangoTIC/ticServer{{< /focus >}} pour lancer la commande, il est possible de créer un [alias](https://doc.ubuntu-fr.org/alias) :
- Ouvrir le fichier .bashrc : {{< cmd >}}nano ~/.bashrc{{< /cmd >}}
- En fin de fichier copier les deux lignes : 
{{< codefile file="/home/pi/.bashrc" lang="bash" >}}
# créer un alias pour la commande npm run dev
alias tailwind-dev='cd /home/pi/djangoTIC/ticServer && npm run dev'
{{< /codefile >}}
- Recharger le fichier {{< focus >}}.bashrc{{< /focus >}} : {{< cmd >}}source ~/.bashrc{{< /cmd >}}
- Depuis n'importe quel dossier on lance la commande : {{< cmd >}}tailwind-dev{{< /cmd >}}
**Quoi qu'il en soit, le plus judicieux est d'ouvrir deux terminaux :**
- un sur lequel on lance {{< focus >}}django{{< /focus >}} pour visualiser le site internet (obligatoirement en environnement virtuel)
- un sur lequel on lance {{< focus >}}tailwind{{< /focus >}} pour observer en direct les modifications éventuelles du css (en environnement virtuel ou non)
### Ajouter un nouveau dossier templates
- Les [fichiers html](https://fr.wikipedia.org/wiki/Hypertext_Markup_Language) doivent être stockés dans un dossier {{< focus >}}templates{{< /focus >}}
- Créer le dossier {{< focus >}}templates{{< /focus >}} : {{< cmd >}}mkdir ~/djangoTIC/ticServer/ticapp/templates{{< /cmd >}}
- Ouvrir {{< focus >}}settings.py{{< /focus >}} en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticServer/settings.py{{< /cmd >}}
- Mettre à jour {{< focus >}}settings.py{{< /focus >}} en indiquant le nouveau chemin des fichiers {{< focus >}}html{{< /focus >}} en l'insérant dans la liste {{< focus >}}TEMPLATES{{< /focus >}} : {{< focus >}}'DIRS': [BASE_DIR / 'templates'],{{< /focus >}}
- Pour aider au débogage lors du développement du site, on peut ajouter dans {{< focus >}}TEMPLATES -> OPTIONS -> context-processors{{< /focus >}} : {{< focus >}}'django.template.context_processors.debug',{{< /focus >}} *(cet ajout n'a pas d'intérêt en phase de production)*
- La liste {{< focus >}}TEMPLATES{{< /focus >}} devient donc :
{{< codefile file="ticServer/ticServer/settings.py" lang="python" >}}
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],                       # new
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',     # new
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
{{< /codefile >}}


