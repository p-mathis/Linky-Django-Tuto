+++
date = '2025-12-18T08:54:18+01:00'
draft = false
h1 = 'Partie 6 : Mise en Prodution'
title = 'Partie 6 : Mise en Production avec Gunicorn et Systemd'
description = "Déploiement du serveur Django en production : configuration de Gunicorn, gestion des fichiers statiques avec WhiteNoise, sécurisation des secrets via un fichier .env et adaptation de settings.py."
+++

{{< line >}}

## Serveur Gunicorn
Actuellement, le site fonctionne en passant par le serveur embarqué de Django en tapant, en environnement virtuel, la commande : {{< focus >}}python manage.py runserver 0.0.0.0:8000{{< /focus >}}.
Il est souhaitable de ne plus passer par ce serveur embarqué et d'avoir son propre serveur. Le choix porte sur [Gunicorn](https://pypi.org/project/gunicorn/)
<!--more--> 
### Installation de Gunicorn
- Se mettre en environnement virtuel : {{< cmd >}}source /home/pi/djangoTIC/venv/bin/activate{{< /cmd >}}
- Installer Gunicorn : {{< cmd >}}pip install gunicorn{{< /cmd >}}
### Créer un service systemd
Ce service lance le serveur *gunicorn*
- Créer en écriture le fichier : {{< cmd >}}sudo nano /etc/systemd/system/django-ticserver.service{{< /cmd >}}
- Copier/coller le code suivant :
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
- *Sauvegarder et quitter*
### Lancer le service
- Recharger et lancer :  
{{< cmd >}}sudo systemctl daemon-reload{{< /cmd >}}  
{{< cmd >}}sudo systemctl enable django-ticserver{{< /cmd >}}  
{{< cmd >}}sudo systemctl start django-ticserver{{< /cmd >}}
- Vérifier le service :  
{{< cmd >}}sudo systemctl status django-ticserver{{< /cmd >}}

#### Mémento de commandes utiles : 

<div class=" items-center  px-2  bg-mycolor-300 text-myred rounded font-mono text-sm border border-mycolor-400">
<span class="text-green-700"># Voir les logs en temps réel </span><br>
sudo journalctl -u django-ticserver -f  <br>
sudo journalctl -u tic-capture -f  <br>

<span class="text-green-700">\# Arrêter/Démarrer </span> <br>
sudo systemctl stop django-ticserver  <br>
sudo systemctl start django-ticserver  <br>

<span class="text-green-700">\# Redémarrer</span><br>
sudo systemctl restart django-ticserver<br>

<span class="text-green-700">\# Désactiver le démarrage auto</span><br>
sudo systemctl disable django-ticserver  
</div>

### Installer WhiteNoise
Avec l'installation de *Gunicorn*, les fichiers statiques ne vont pas nécessairement être servis correctement. [Whitenoise](https://whitenoise.readthedocs.io/en/latest/) permet de pallier ce problème.
#### Installation
- Se mettre en environnement virtuel
- Installer {{< focus >}}whitenoise{{< /focus >}} : {{< cmd >}}pip install whitenoise{{< /cmd >}}
#### Modifier settings.py
- Ouvrir le fichier en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticServer/settings.py{{< /cmd >}}
- Chercher la liste {{< focus >}}MIDDLEWARE{{< /focus >}}
- Ajouter : 
{{< codefile file="/etc/systemd/system/django-ticserver.service" lang="ini" >}}
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # ← Ajouter cette ligne juste après SecurityMiddleware
    'django.contrib.sessions.middleware.SessionMiddleware',
    # ... le reste des middlewares
]
{{< /codefile >}}
- Au niveau de {{< focus >}}STATIC_URL = 'static/'{{< /focus >}}
- Ajouter deux entrées supplémentaires : 
{{< codefile file="/etc/systemd/system/django-ticserver.service" lang="ini" >}}
STATIC_URL = '/static/'     # conserver cette ligne

# à ajouter

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')  
STORAGES = {
    "default": {
        "BACKEND": "django.core.files.storage.FileSystemStorage",
    },
    "staticfiles": {
        "BACKEND": "django.contrib.staticfiles.storage.StaticFilesStorage",
    },
}
{{< /codefile >}}
- *Enregistrer et fermer*
- Relancer le serveur : {{< cmd >}}sudo systemctl restart django-ticserver{{< /cmd >}}
{{< line >}}
## Passer en production
En développement, le paramètre {{< focus >}}DEBUG{{< /focus >}} de {{< focus >}}settings.py{{< /focus >}} est fixé à {{< focus >}}True{{< /focus >}}. Cela permet de voir rapidement un dysfonctionnement au cas où une erreur serait renvoyée par le serveur.  
Si on ne passe pas la valeur de {{< focus >}}DEBUG{{< /focus >}} à {{< focus >}}False{{< /focus >}}, un risque d'intrusion dans le réseau local existe, surtout si on décide d'accéder aux données du Linky depuis l'extérieur du réseau. À la limite, la Raspberry peut être détournée de sa fonction initiale ; un code malveillant peut être installé et la Raspberry servir de relais à d'autres attaques.
### Créer une nouvelle *SECRET_KEY*
- Installer {{< focus >}}python-decouple{{< /focus >}} en environnement virtuel :{{< cmd >}}pip install python-decouple{{< /cmd >}}
- Créer une nouvelle clé : {{< cmd >}}python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'{{< /cmd >}}

### Créer un fichier d'environnement
- Afin de ne pas coder en dur la nouvelle clé {{< focus >}}SECRET_KEY{{< /focus >}} qu'on va créer, on la copie dans un fichier {{< focus >}}.env{{< /focus >}} à la racine du site
- Créer et ouvrir en écriture {{< focus >}}.env{{< /focus >}} : {{< cmd >}}nano ~/djangoTIC/ticServer/.env{{< /cmd >}}
- Écire le fichier :
{{< codefile file="djangoTIC/ticServer/.env" lang="bash" >}}
SECRET_KEY=votre-cle-generee-ici
DEBUG=False
ALLOWED_HOSTS=192.168.x.x,localhost
{{< /codefile >}}
- **Aucun espace avec les = ou avec les virgules**
- Remplacer x.x par l'adresse IP de la Raspberry (par exemple : 192.168.1.120)
### Modifier *settings.py*
- Ouvrir en écriture : {{< cmd >}}nano ~/djangoTIC/ticServer/ticServer/settings.py{{< /cmd >}}
- Ajouter ou modifier les éléments suivants : 
{{< codefile file="ticServer/ticServer/settings.py" lang="python" >}}
from decouple import Config, RepositoryEnv    #pour la production et le masquage des données
# Spécification du chemin exact de .env
config = Config(RepositoryEnv(BASE_DIR / '.env'))
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=lambda v: [s.strip() for s in v.split(',')])
{{< /codefile >}}
- Le fichier {{< focus >}}settings.py{{< /focus >}} final est le suivant :
{{< codefile file="djangoTIC/ticServer.env" lang="bash" >}}
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


### Reconfigurer les fichiers statiques
- Après ces modifications, il faut collecter les fichiers statiques
- Dans {{< focus >}}~/djangoTIC/ticServer{{< /focus >}} lancer la commande : {{< cmd >}}python manage.py collectstatic \-\-noinput{{< /cmd >}}
- Cette commande va créer un dossier {{< focus >}}staticfiles{{< /focus >}} à la racine du site. C'est ce dossier qui va servir les fichiers statiques au site en production.  
- Relancer le serveur : {{< cmd >}}sudo systemctl restart django-ticserver{{< /cmd >}}
{{< line >}}
{{< gallery images="tic_jour.jpg, tic_hier.jpg, tic_lasthour.jpg, tic_last24hours.jpg" >}}
