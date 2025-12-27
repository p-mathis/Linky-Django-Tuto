+++
date = '2025-12-15T11:20:34+01:00'
draft = false
title = 'Tutoriel Linky'
description = "Tutoriel pour capturer avec une raspberry pi les données téléinformation client (TIC) d’un compteur linky, les stocker dans une base de données et les lire sur un serveur Django"
+++

{{< line >}}

Les compteurs Linky disposent d'une fonction de TéléInformation Client (TIC), qui va donner en continu un certain nombre d'informations, notamment la puissance apparente instantanée soutirée, les index d'énergie consommée.  
Ces informations sont directement lisibles sur le compteur Linky lui-même, mais cette lecture est, à l'évidence, malaisée. Le but de ce tutoriel est de mettre en place un système de lecture fiable et confortable.
{{< line >}}
## Objectif
- Capturer les données fournies en continu par le compteur Linky
- Stocker les valeurs obtenues dans une base de données
- Visualiser à distance ces données par le biais d'un site internet
{{< line >}}
## Matériel
### Raspberry Pi
- La Raspberry Pi munie d'une carte micro-SD permet de stocker les données et d'héberger le site internet
- Cette Raspberry n'a pas besoin d'être particulièrement puissante pour le captage des données
- Mais dans la mesure où on va installer dessus un serveur avec Django, Tailwind, Flowbite et Gunicor, une configuration confortable nécessite au minimum une Raspberry Pi3 B+
- Pour ce tutoriel nous avons utilisé une Raspberry Pi 4B
- Les Raspberry sont disponibles sur de multiples plateformes, mais notamment chez le fournisseur officiel [Kubii](https://www.kubii.com/fr/)
### La carte SD
- On peut estimer que les données récoltées par la base de données seront de l'ordre de 1 à 2 Go par an
- Le problème de la taille de la carte n'est pas tant lié à sa capacité de stockage qu'à l'usure des blocs
- Il est donc conseillé de prendre une carte de 32 Go, idéalement 64
- Surtout il convient d'avoir une carte de [type industrielle ou High Endurance](https://www.kingston.com/fr/memory-cards?card%20type=microsd)
- Une alternative : utiliser un disque SSD de petite capacité avec un boîtier externe et faire le boot directement depuis le SSD ; la longévité du système est nettement augmentée
- Dans le cas présent, une carte SD 64Go a été utilisée
### L'optocoupleur
- L'[optocoupleur](https://fr.wikipedia.org/wiki/Photocoupleur#:~:text=Un%20photocoupleur%2C%20ou%20optocoupleur%2C%20est,l'anglais%20optocoupler%20ou%20optoisolator.) récupère les données de la TIC
- Plutôt que de monter soi-même son propre optocoupleur, le plus simple est d'acheter un module (shield) déjà monté / soudé. 
- Le module [PiTinfo](https://hallard.me/pitinfov12-light/) est disponible pour un coût modeste chez [Tindle](https://www.tindie.com/products/hallard/pitinfo/) et fonctionne parfaitement

### Câble
- Câble à deux conducteurs pour relier l'otpocoupleur au compteur Linky
- Si possible blindé et torsadé pour atténuer les parasites, catégorie [Cat 5](https://fr.wikipedia.org/wiki/C%C3%A2ble_cat%C3%A9gorie_5) ou [Cat 6](https://fr.wikipedia.org/wiki/C%C3%A2ble_cat%C3%A9gorie_6)

{{< line >}}
## Résultats
- Le but est d'afficher diverses données sur des pages internet
- Par exemple : l'intensité instantanée soutirée, phase par phase si on est en triphasé : 
<div class="w-full md:w-2/3 lg:w-1/2 xl:1/3 pt-4 md:pt-8">
{{< imgResize src=tic_instant.jpg alt="consommation instantanée en watt sur un compteur linky" >}}
</div>