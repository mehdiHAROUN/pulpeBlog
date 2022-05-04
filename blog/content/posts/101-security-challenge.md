---
title: "101 Security Challenge"
date: 2022-04-13T13:55:59+02:00
draft: true
---

# Injection sql
- http://sqlmap.org/
- https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet

la solution magique de base : ' or 1=1--
http://workitniveau1.chall.malicecyber.com/
http://workitniveau22.chall.malicecyber.com/
e.alderson19860917



# Injection de commande 
- http://www.openvas.org/
- https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project
- https://www.owasp.org/index.php/OS_Command_Injection_Defense_Cheat_Sheet

https://online.malice.fr/#/ci/TUc4xpXbFyas9bAVd72J/retineye-niveau-1/
http://manpages2.chall.malicecyber.com/
https://retineyeniveau12.chall.malicecyber.com/


``` 
String[] cmdArray = new String[4];
cmdArray[0] = "sh";
cmdArray[1] = "-c";	
cmdArray[2] = "man ";
cmdArray[3] = manpage;
```

Process process = Runtime.getRuntime().exec( cmdArray );


OWASP Broken Access Control https://owasp.org/index.php/Broken_Access_Control
OWASP Proactive Controls: Access Controls https://www.owasp.org/index.php/OWASP_Proactive_Controls#6:_Implement_Access_Controls
OWASP Application Security Verification Standard: V4 Access Control https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project#tab.3DHome
OWASP Testing Guide: Authorization Testing https://www.owasp.org/index.php/Testing_for_Authorization
OWASP Cheat Sheet: Access Control https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html


## Local File Disclosure (LFD)
http://retineyeniveau22.chall.malicecyber.com/

https://retineyeniveau22.chall.malicecyber.com/view.php?rub=etc&file=passwd
http://retineyeniveau32.chall.malicecyber.com/

https://exiftool2.chall.malicecyber.com/?image=cat


## XSS  

Définition
Le Cross-Site Scripting (abrégé XSS) est un type de vulnérabilité de sécurité qui touche les sites Web et permet à un attaquant d'injecter du contenu dans une page, provoquant ainsi des actions sur les navigateurs Web des clients la consultant.

Il existe deux types de XSS :

XSS réfléchie : L'application ne contrôle pas les entrées utilisateur et les restituent sur la page HTML telles que fournies par ce dernier. Une attaque réussie peut permettre à l'attaquant de faire interpréter du code HTML et/ou JavaScript arbitraire dans le navigateur de la victime. Typiquement, la victime devra interagir avec un lien malveillant qui pointe vers une page contrôlée par l'attaquant.

XSS stockée : L'application conserve les données d'entrée non validées et non échappées de l'utilisateur en base de données. Celles-ci sont ensuite restituées dans la page Web pour chaque utilisateur consultant cette dernière. La vulnérabilité XSS stockée est souvent considérée comme un risque élevé ou critique, car elle permet potentiellement de conduire une escalade de privilège si la victime possède plus de droits que l'attaquant. De plus, cette attaque est persistante contrairement à la XSS réfléchie.

Note: Il existe d'autres types de vulnérabilités XSS.

Les attaques XSS permettent de réaliser les actions suivantes :

Le vol de session ;
L'usurpation de compte ;
Téléchargement de logiciels malveillants sur l'ordinateur de la victime pour prendre le contrôle de ce dernier ;
Enregistrement des frappes du clavier de la victime ;
Ingénierie sociale : redirection vers une page sous le contrôle de l'attaquant afin de voler des identifiants (Facebook, Gmail, LinkedIn ...) ;
Etc...


http://sitevunerable.com/index.php?name=<script>alert('XSS')</script>


Les outils automatisés peuvent détecter et exploiter toutes les formes de XSS, et il existe des frameworks disponibles gratuitement (OpenVAS, OWASP ZAP).

http://www.openvas.org/
https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project

http://internalsupportniveau1.chall.malicecyber.com/login/?next=%2F
https://requestbin.com/
