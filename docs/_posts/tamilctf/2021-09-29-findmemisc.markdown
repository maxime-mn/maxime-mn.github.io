---
layout: post
title:  "Find Me (Misc)"
date:   2021-09-29
categories: tamilctf
---
# Introduction

Ce challenge de la catégorie misc était le plus difficile considérant le nombre de points qu'il donnait.

# Résolution

Le challenge comporte une simple image.

![pexels_beta](https://user-images.githubusercontent.com/16634117/135309770-65a9762c-9e8d-4afd-bfcc-3ba0f75a2ae4.jpg)

Dans cette image se trouve un hint qui nous dit de vérifier l'exif. Je me rends donc sur le site `http://exif.regex.info/` qui permet d'extraire l'exif d'une image en ligne. Le résultat est le suivant : 

![Screenshot_2021-09-29_07-57-14](https://user-images.githubusercontent.com/16634117/135310066-2b42e9a5-6a71-4e3d-97a5-31fa0f7ba129.png)

![Screenshot_2021-09-29_07-57-30](https://user-images.githubusercontent.com/16634117/135310068-0eab9fb7-94d8-4f98-92a7-4930b4dcd9b7.png)

On remarque que le flag présent est un rabbit hole et qu'il ne semble pas y avoir de donnée utile sauf le commentaire du fichier qui est étrange.

J'ai testé différentes actions réflexes sur le fichier : binwalk, bit de poids faible, strings, etc. Mais en vain.

En regardant l'image de plus près j'ai remarqué différents mots dispersés partout sur l'image, j'ai remarqué que chaque mot avait une longueur différente, j'ai mis cette information en lien avec le commentaire de l'image, la suite de nombre représente peut être le mot de la taille du nombre qu'il faut prendre.


En concaténant le mot de taille 8, puis de taille 5 puis 7, etc. : `You_cant_GET!ForEver007700`

Sur l'image se trouve également les chaines `TamilCTF` et `}`

Je concatène le tout : `TamilCTF{You_cant_GET!ForEver007700}` bingo !
