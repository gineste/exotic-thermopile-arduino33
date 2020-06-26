# Module Thermopile

## Principe

Ce wiki traite de l'utilisation de la thermopile. Une thermopile est un dispositif électronique qui convertit l'énergie thermique en énergie électrique. Elle est composée de plusieurs thermocouples connectés généralement en série ou, plus rarement, en parallèle. Un tel dispositif fonctionne selon le principe de l'effet thermoélectrique, c'est-à-dire qu'il génère une tension lorsque ses métaux différents (thermocouples) sont exposés à une différence de température. Ici la différence de température est celle entre l'arrière de la thermopile (à la température ambiante) et celle faisant face à l'objet à mesurer dont on reçoit le rayonnement.

![image_composants](therm.png)

## Spécifications

Les thermopiles se différencient entre autres par leur champ de vue et leur gamme de température. Nous avons sélectionné un modèle à faible champ de vue et une gamme de températures ambiantes (0 à 85°C). 

| Fabricant   | Texas Instruments           |
| ----------- | --------------------------- |
| Modèle      | **OMRON D6T-1A-01**         |
| Temp range  | -20°C à 80°C                |
| angle 50%   | 58°                         |

Il s'agit d'un modèle pour intégration industrielle, donc c'est un capteur seul qu'il a fallu packager dans une carte électronique et un boitier ExoticSystems customisé. 

## Montage et composants

Le bloc thermopile est composé des éléments suivants:
- boitier baliz Exotic Systems
- Thermopile
- Carte ARDUINO NANO 33 IOT
- Laser de pointage 5mW (modèle pour arduino ANGEEK XF0085X5 650nm)
- Connecteur micro-USB à l'arrière

Le laser de pointage est monté dans le même axe vertical que la thermopile, 20mm plus bas, et est matérialisé par un clignotement du pointeur rouge. Il est recommandé d'indiquer un risque pour les yeux (laser classe IIIA).


## Positionnement: Il n'y a pas de seuil de distance d'utilisation, seul la contrainte de largeur de champ prévaut. L'angle de champ est de 58° et donne donc, pour différentes distances, un champ en largeur de:

| distance à la cible | largeur de perception pour 90% du flux |
| ------------------- | -------------------------------------- |
| 20cm                | 22cm                                   |
| 35cm                | 38cm                                   |
| 60cm                | 66cm                                   |

On voit donc qu'on peut éloigner la thermopile jusqu'à 35cm de sa cible, ensuite on débordera sur les côtés du KM et on mesurera alors également le support ou des objets avoisinants. On recommande donc de bien centrer la thermopile au dessus du KM, ou de la rapprocher. 

![image_device](therm3.png)

## Acquisition et script

L'acquisition des données de la thermopile se fait via le port série, et donc le script **serialObject.py** qui définit une classe permettant de faire communiquer nos objets en USB.

La thermopile renvoie ses données soit à intervalle constant (exemple 1seconde) soit lorsqu'on envoie une commande `waitResponseWithData`. Les données sont sous forme de string comprenant deux valeurs de température avec un chiffre après la virgule, la seconde étant celle d'intérêt:

| Température | Explication                                           |
| ----------- | ----------------------------------------------------- |
| PTAT        | Température du thermocouple interne en degrés celsius |
| THPI        | Température de l'objet distant en degrés celcius      |

Le temps nécessaire pour la communication et la réception des données est inférieur à **20ms**. Chaque requete `waitresponsewithdata()` execute un `readline()` puis décode les bytes en deux temps: la première mesure correspond aux données PTAT, la seconde aux données THPI. L'envoi des données en mode push se fait toutes les 500ms, ce qui fait 1000ms pour les deux requêtes (PTAT et THPI). Ce temps peut être paramétré plus court (200ms). Toutefois, la classe `serialObject` définit un nouveau thread donc l'attente de la valeur se fait en parallèle des instructions suivantes du scriptprises de mesure de vision, par exemple. Il est donc plus efficace d'acquérir ce capteur en premier.

`response = self._serialport.readline().decode("utf-8")`

## Qualité et robustesse de mesure

Les tests de robustesse sur le banc de test ont pu montrer que:

- La documentation technique ne fournit pas de correction pour le facteur d'émissivité, et on fait l'hypothèse qu'il est corrigé automatiquement ou négligeable.
- La température mesurée par la thermopile est dans tous les cas exacte à +/-1°C de la vraie température en étuve. 
- Un éclairage direct fort sur la cible va modifier sa température de surface et être détecté par la thermopile (la température de surface serait alors plus élevée que la température interne du KM)
- Un éclairage fort (type halogène) vers la thermopile va fausser les mesures à la hausse.