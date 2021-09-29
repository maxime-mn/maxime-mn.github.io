---
layout: post
title:  "CringeNcoder"
date:   2021-09-29
categories: tamilctf
---

# Introduction

Ce challenge de la catégorie Web était le plus simple considérant le nombre de points qu'il rapportait. Je l'ai donc testé en premier.

## Résolution

Au début du challenge, j'ai simplement une page dans laquelle je peux taper du texte pour le transformer en "cringe langage". 


![1](https://user-images.githubusercontent.com/16634117/135307400-a482a4cb-fbb6-4692-b013-42d0aeb7b3a5.png)

Comme à chaque CTF web je vérifie s'il n'existe pas un endpoint `/flag`
![2](https://user-images.githubusercontent.com/16634117/135307510-aa2bacb6-b839-4c15-a95f-376129c890d0.png)

J'ai donc un texte écrit dans ce language et je dois retrouver l'original. La page donne un indice utile, le flag ne peut être composé que de caractères minuscules et de chiffres. Je tape donc la chaine `abcdefghifjklmnopqrstuvwxyz0123456789` afin d'obtenir l'équivalent _cringe_ de chaque caractère.

Une fois cela fait, j'ai écrit un script python permettant de lier la valeur _cringe_ à son équivalent en décodé puis de récupérer le flag en prenant l'équivalent de chaque valeur _cringe_ dans mon dictionnaire.

```python
alphabet = "abcdefghijklmnopqrstuvwxyz0123456789"
vals = "cringe cr1nge cRinge crIng3 cRimG3 cR1Ng3e criNgee CRINGE crinGE ccR1nge CriNGE cRINGe cr1ngE cringE CRIng3 Cr1nGe cR1nnge cR1Ng3 CrInGe cRingE cR1NGE CRiNg3 CRINGe CR1NGe cring3 CRIMNGE cRInGE crinG3 cRInge cRinG3 criNG3 cr1NG3 crinGe cRiNge CrInGE CRinGE"

alphabet =  [word for word in alphabet]
vals = vals.split(' ')

dic = dict(zip(vals, alphabet)) 

encoded = "cR1Ng3e crinG3 cringE cringe cRINGe cRINGe cring3 crinG3 cringE cringE cRinG3 Cr1nGe cRimG3 criNG3 cRinge cRimG3 cringe cR1Ng3e cRiNge cRinG3 cR1Ng3 CrInGe cRInGE cr1ngE criNG3 cringE cring3 cRinge cR1Ng3 crinGE cringE criNgee cRinG3 CrInGe"

encoded = encoded.split(' ')
sol = ''
for val in encoded :
    sol += dic[val]

print(sol)

```

Mon script me donne la chaine `f1nally1nn3pe4ceaf73rs0m4nycring3s` qu'il ne me reste plus qu'à ajouter entre TamilCTF{} pour valider le challenge.

