---
title: "Store Json Database"
date: 2022-11-21T10:30:53+01:00
draft: false
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

## Intro

Entity Framework 7 apporte la gestion complète de la colonne Json:

.NET 7 est là ! qui embarque une nouvelle version de l'Entity Framework 7 l'ORM (object relational mapping) le plus populaire des développeur .net.
Parmi les nouvelles fonctionnalités qu'on va explorer dans cet article est la manipulation des colonnes de type JSON.
De nos jours la plupart des sgbd supporte des colonnes json ce qui donne cette dimension document à une bdd relationnelle.
EF7 implémente le fournisseur SQl server qui permet de faire des requêtes Linq nécessaire pour l'exploration et la modification des documents JSON.

## Mappage à des colonnes JSON

## Requêtes dans des colonnes JSON

## Mise à jour des colonnes JSON
