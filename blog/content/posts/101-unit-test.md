---
title: "101 Unit Test"
date: 2022-02-21T13:35:52+01:00
draft: false
---

Pourquoi tester ? test c'est douter !

Type de test:
tests unitaires : test bas niveau pour tester une seule unité | exécuté en ms
tests d'intégration : interaction des services (BDD, Api) 
tests fonctionnels : interagir avec un service selon les exigences métiers
tests de bout en bout (test automatiques) : reproduire le comportement d'un utilisateur final dans un environnement de test | à automatisé | plusieurs minutes   
tests d'acceptation : test de PO
tests de performance : vérifier le comportement en cas de surcharge
tests de sécurité : OWASP

le coût des tests :




Les tests unitaires, pourquoi ? 

Tester plusieurs scénarios d'un même fonctionnement.
Factoriser le code.
Assurer la non regression.


Pourquoi on aime pas les tests unitaires ?
ça demande une certaine rigueur.
ça prend du temps (on oublie de les estimer) 
L'architecture du code ne facilite pas l'écriture des tests (Interface, static )
Il faut les maintenir à jour.


Couverture de code , pourquoi ? :
une indication quantifiable sur le degré de test.

Comment je fais pour le savoir ? 
Via visual studio
Via sonar cloud

Attention !!! une bonne couverture ne veut pas dire que le code est bien testé.







