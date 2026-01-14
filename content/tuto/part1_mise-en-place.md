+++
date = '2025-12-15T22:36:31+01:00'
draft = false
title = 'Partie 1 : Mise en place'
description = "Activation du ssh sur une raspberry pi ; mise en place d’un module (shield) PiTinfo sur le GPIO ; connexion au compteur Linky  avec utilisation du mode standard"
+++

{{< line >}}

## Préparer la Raspberry
- La Raspberry doit être physiquement à côté du compteur Linky, puisque qu'elle porte le module [PiTinfo](https://hallard.me/pitinfov12-light/) et que celui-ci doit être relié par câble aux bornes TIC du compteur.
- Il faut donc préparer la Raspberry pour communiquer avec elle en [ssh](https://fr.wikipedia.org/wiki/Secure_Shell) depuis son propre ordinateur.
### Installer Raspbian sur la Rasbperry
- On suit les instructions du [tutoriel Raspberry](https://raspberrytips.fr/installer-raspberry-pi-os/)
- Sur son ordinateur on installe [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- On flashe la carte SD
- Une fois la carte prête, on l'insère dans la Raspberry **éteinte**
- On branche un clavier, une souris, un écran au niveau de la Raspberry
- On allume la Rasbperry
- On procède à l'installation en introduisant les différents paramètes souhaités
### Activer le ssh
- C'est une étape fondamentale, puisque toute la mise en place des différents codes va se faire en ssh
- Le mot de passe de la Raspberry doit être **consistant**
- Il est préférable de changer le port ssh externe au niveau du routeur, de manière à éviter les tentatives d'intrusion externe par le port par défaut
- Il est utile de donner une IP locale fixe à sa Raspberry, en sachant que la Raspberry a deux IP : une pour le WiFi, une pour l'éthernet
- Des conseils sont disponibles sur un tutoriel de [mise en place de caméras de suveillance](https://djangocamera.netlify.app/tuto/part1/#modifications-au-niveau-de-la-box)
### Débrancher la Raspberry
- Le ssh étant activé, on peut débrancher la Raspberry

{{< line >}}
## Préparer le module PiTinfo
### Où le connecter ?
- Au niveau du compteur Linky, la connexion est différente selon que l'installation est en triphasé ou en monophasé
- La prise est accessible en enlevant le capot *ad hoc* **non plombé**
- En monophasé, la prise TIC est située en bas du compteur (le compteur a deux capots : le supérieur plombé, l'inférieur non plombé)
- En triphasé, la prise TIC est située en haut du compteur (le compteur a trois capots : le supérieur est l'inférieur non plombés, le médian plombé)
- Cette page internet permet de visualiser l'[ouverture de son compteur Linky](https://support.ecojoko.com/hc/fr/articles/8957701273116-Ouvrir-le-capot-de-mon-compteur-Linky)
- Les bornes de connexion sont nommées : **I1** et **I2**
### Comment le connecter ?
- Utiliser un cable à deux conducteurs, si possible blindé de manière à avoir un signal *propre*
- Fixer la première extrêmité des conducteurs au niveau des bornes de la prise TIC
- Fixer la deuxième extrêmité des conducteurs au niveau des bornes du module PiTinfo
- Il n'y a pas de polarité et donc peu importe que tel conducteur soit lié à telle borne
- Bien assurer les fixations
### Positionner le module PiTinfo sur la Raspberry
- Le module PiTinfo a un connecteur à 5 doubles entrées qui se fixe sur le GPIO de la Raspberry
- Bien le positionner sur les 5 premières double broches du GPIO en s'assurant de couvrir les broches TXD et RXD
<div class="w-2/3 md:w-1/3 pt-4 pl-2">
{{< imgResize src=raspberry-pi-pinout.jpg alt="images des bornes gpio d'une raspberry" >}}
</div>

- L'image de positionnement de [Tindie](https://cdn.tindiemedia.com/images/resize/IwpGpgtxxo9LO_IxsfZaK9GIRvk=/p/full-fit-in/1782x1336/i/5857/products/2020-07-23T18%3A23%3A11.467Z-IMG_7296..jpeg?1606306133) permet de bien repérer celui-ci
<div class="w-2/3 md:w-1/3 pt-4 pl-2">
{{< imgResize src=tic_pitinfo_GPIO.jpg alt="images des bornes gpio d'une raspberry" >}}
</div>

{{< line >}}
## Brancher la Raspberry
- Une fois toutes les connexions réalisées, brancher la Raspberry
- Cette installation étant réalisée, toutes les autres opérations se font depuis un ordinateur 

{{< line >}}
## Faire mettre le Linky en mode standard
- Il existe deux modes de fonctionnement d'un compteur Linky : historique et standard
- Pour obtenir les bonnes informations, le Linky doit être en mode standard
- Il est possible de savoir en quel mode fonctionne le compteur Linky en faisant défiler les informations au niveau de compteur et en regardant si il affiche {{< focus >}}Mode Historique{{< /focus >}} ou {{< focus >}}Mode Standard{{< /focus >}}
- La demande de passage du mode historique au mode standard se fait en contactant son fournisseur d'électricité ou bien le gestionnaire du réseau, [Enedis](https://www.enedis.fr/)
- Cette page internet donne des informations sur les modes [historique et standard](https://support.ecojoko.com/hc/fr/articles/9077627829020-Passage-du-compteur-Linky-en-mode-standard)