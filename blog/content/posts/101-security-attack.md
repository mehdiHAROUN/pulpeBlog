---
title: "101 Security Attack"
date: 2022-09-21T14:07:12+02:00
draft: false
---

boite noir / gris (url + credentials) / blanche (architecture)

scénario :
app web : owasp / PTES (infra)
PCI DSS
PASSI

Score CVSS (0-10) : évaluer la criticité des vulnérabilités

bind shell : il faut une adresse ip pour exec du code dans la machine
reverse shell : la machine se connecte sur ma machine

Pour attaquer :
1- recherche passive : shodan | google dorks | https://gist.github.com/sundowndev/283efaddbcf896ab405488330d1bbc06
2- recherche active : Nmap , Nessus , Nikito , wpscan | whois | webappalizer |falco container
gophish (phishing , on peut utiliser spf(quelle @ ip peut envoyer un meail avec un nom de domaine), dkim)
