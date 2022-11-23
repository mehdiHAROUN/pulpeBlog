---
title: "Store Json Database"
date: 2022-11-21T10:30:53+01:00
draft: true
---

trois types de base de donnée :
bdd relationnel
bdd document
bdd graph

Avantage de stocker la data en json :

- simple et sans surcoût en terme de code lorsque on intègre des données en json comme des logs.
- stocker des informations avec une structure qui change souvent comme le paramétrage d'un écran ou des fonctionnalités
- par soucis de performance pour récupérer les données avec un seul appel sans passer par plusieurs tables

Les inconvénients :

- l'intégrité de l'informations, possibilité d'avoir la meme donnée dans plusieurs endroits
- complexité de la requête NoSql

une solution peut réconcilier les deux méthodes, c'est d'avoir un type Json dans la bdd (ce qui est le cas sur quelques sgbd depuis 2015 deja).

Entity Framework 7 apporte la gestion complète de la colonne Json.
