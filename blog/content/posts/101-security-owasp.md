---
title: "101 Security Owasp"
date: 2022-04-15T14:18:45+02:00
draft: false
---

## INJECTION SQL

Définition
Les vulnérabilités d'injection se produisent lorsque des données non fiables sont envoyées à un interpréteur dans le cadre d'une commande ou requête. Les données malveillantes de l'attaquant peuvent amener l'interpréteur à exécuter des commandes involontaires ou à accéder à des données sans autorisation appropriée.

Les failles d'injection peuvent être de plusieurs types. Les principaux sont les suivants :

Injection SQL ;
Injection de commandes ;
Injection NoSQL ;
Injection LDAP.

Une application est vulnérable aux injections lorsque :

Les données fournies par l'utilisateur ne sont pas validées, filtrées ou échappées par l'application ;
Les entrées sous le contrôle des utilisateurs sont utilisées directement dans l'interpréteur ;
Les données malveillantes sont directement utilisées ou concaténées, de sorte que la requête SQL contient à la fois des données de structure et des données hostiles dans des requêtes dynamiques, des commandes ou des procédures stockées.
Impacts
Les impacts liés aux vulnérabilités d'injection sont multiples :

Récupération du contenu de la base de données ;
Contournement d'un formulaire d'authentification ;
Suppression de données en base, interruption du service ;
Exécution de code arbitraire à distance ;
Compromission totale ou partielle du serveur hébergeant l'application vulnérable.

Afin d'appréhender au mieux le concept de vulnérabilités d'injection, nous étudierons le cas des injections SQL et injections de commandes.

image1.png

Exploitation
Insertion du code malveillant dans l'URL ou dans le formulaire :

Il est alors possible d'insérer une charge utile malveillante directement dans l'URL (méthode GET utilisée) afin de récupérer des informations contenues dans la base de données telles que le nom de l'utilisateur courant et le nom de la base de données courante:
id = -1' union select user(),database() -- -
http://site_vunerable.com/index.php?id=-1' union select user(),database() -- -
En substituant la valeur de la variable id, on obtient la requête suivante :
$query = "SELECT first_name, last_name FROM users WHERE user_id = '-1' union select user(),database() -- -';";

image 2

Exploitation
Il est alors possible d'insérer des commandes dans le formulaire (en POST) qui seront directement envoyées à la méthode shell_exec() puis exécutées par le système.

Comme le résultat de la requête est affiché à l'écran, nous avons placé une charge malveillante afin de récupérer le nom de l'utilisateur courant ainsi que des informations relatives au noyau du système d'exploitation installé.

image 3

L'examen du code source est la meilleure méthode pour détecter si les applications sont vulnérables aux injections, suivi de près par des tests automatisés approfondis de tous les paramètres, en-têtes, URL, cookies, JSON, SOAP et données XML entrées.

https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project

Des outils automatisés peuvent détecter et exploiter toutes les formes de vulnérabilités d'injection. Certains de ces outils permettent d’identifier plusieurs types d'injection, alors que d'autres sont spécifiques à un seul type.

Pour le cas de l'injection SQL, l'outil le plus connu est SQLMap. Il est capable de détecter et d'exploiter presque toutes les injections SQL pour de nombreux systèmes de gestion de base de données : MySQL, MariaDB, MS-SQL, PostegreSQL, Oracle.

http://sqlmap.org/

Les injections de commandes peuvent par ailleurs être identifiées au moyen de scanner de vulnérabilités disponibles librement, tels que OpenVAS, ou encore ZAP OWASP.

http://www.openvas.org/
https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project

Concernant les injections SQL, plusieurs solutions permettent de se protéger contre ce type d'attaque. Le premier axe de défense concerne l'emploi d'un ou plusieurs mécanismes de protection visant à désamorcer la charge utile soumise par l'attaquant:

Les requêtes préparées ;
Les procédures stockées ;
Les listes blanches ;
Le filtrage ou l'échappement de toutes les entrées sous le contrôle de l'utilisateur.

Enfin, il convient d'appliquer le principe de défense en profondeur et de respecter le principe de moindre privilège.

Durcissement de la configuration du service de gestion de base de données.

Plus d'informations sur le sujet sont disponibles à cette adresse :

https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet

Concernant les mécanismes de protection liés à la vulnérabilité d'injection de commandes, il est recommandé de respecter les instructions suivantes :

Éviter autant que faire se peut les appels système prenant en argument une entrée utilisateur ;
Échapper les entrées utilisateur en considérant le système d'exploitation présent ;
Paramétrage en conjonction avec la validation des données d'entrée.

Plus d'informations sur le sujet sont disponibles à cette adresse :

https://www.owasp.org/index.php/OS_Command_Injection_Defense_Cheat_Sheet

## VIOLATION DE CONTRÔLE D’ACCÈS

Définition
Les contrôles d'accès sont conçus pour gérer et restreindre aux utilisateurs les autorisations sur les fonctionnalités et le contenu d'une application.
Ils permettent de mettre en œuvre les matrices d'accès associant privilèges et rôles d'utilisateurs.

Un problème de contrôle d'accès se pose lorsqu'un un attaquant réussi à récupérer ou modifier des informations qui lui sont normalement inaccessibles à partir de son profil d'utilisateur actuel.
Impacts
Ce type de vulnérabilité a généralement pour conséquence une fuite d'information.

Voici des exemples concrets d'exploitations par un attaquant :

- obtention d'un accès non autorisé au niveau fonctionnel (des API ou des fonctionnalités cachées) ;
- gain d'accès à des données sur le serveur (PATH traversal, LFI : Local File Inclusion, LFD : Local File Disclosure) ;
- élévation de privilège horizontale, soit la possibilité d'agir comme un autre utilisateur de profil identique et de voir, modifier ou supprimer ses données ;
- élévation de privilège verticale, soit la possibilité d'agir comme un utilisateur plus privilégié (un administrateur, par exemple) ;
- contournement de pages soumises à une authentification ou un accès privilégié ;
- accès direct à une ressource non sécurisée (IDOR: Insecure Direct Object References).

### Insecure Direct Object References (IDOR)

Définition
Une vulnérabilité de type Insecure Direct Object References est présente dans une application quand un utilisateur peut accéder à une ressource, normalement inaccessible, en effectuant directement une requête vers celle-ci.
Il peut ainsi passer au travers des différents mécanismes d'authentification et de vérification des permissions.
Scénario : L'application ne vérifie pas les droits pour accéder aux informations d'un compte
Dans ce scénario, nous utilisons le site https://example.com et les informations sur nos abonnements sont accessibles à l'adresse suite :
https://example.com/auth/user/subscription/?account=1337
Nous pouvons identifier que la valeur du paramètre account permet d'obtenir nos informations. Généralement, dans ce type de configuration, cette valeur est attachée à notre compte et elle est incrémentée à chaque création d'un nouveau compte sur l'application.

La valeur du paramètre account étant prévisible, si l'application ne vérifie pas correctement les droits d'accès à cette ressource, nous avons accès aux données d'un autre utilisateur (ici celui avec l'identifiant 42) :
https://example.com/auth/user/subscription/?account=42

### Local File Disclosure (LFD)

Définition
Une application vulnérable à une faille de sécurité de type Local File Disclosure ne met pas en place assez de contrôle et de filtres lors de l'affichage ou du téléchargement d'une ressource.
Un attaquant peut ainsi récupérer une ressource du serveur Web ou du système grâce à une fonctionnalité de l'application.
Scénario : Divulgation de fichiers locaux en raison d'un manque de filtrage des paramètres utilisateurs
Le site web https://example.com possède une fonctionnalité permettant de télécharger une ressource de l'application. Pour exécuter cette fonctionnalité, le client effectue une requête à cette adresse :
https://example.com/auth/user/download.php?id=1
Le paramètre id permet d'indiquer quelle ressource doit être récupérée par le serveur, désignée par une valeur entière.
Cependant, le paramètre souffre d'un manque de filtrage et se trouve ainsi être vulnérable à une faille de sécurité de type LFD.

En effet, lors de l'exploitation, l'attaquant peut envoyer une chaine de caractères.

Par exemple, il peut remplacer la valeur de l'identifiant id par un chemin relatif vers le fichier /etc/passwd :
https://example.com/auth/user/download.php?id=../../../../etc/passwd
Nous obtenons ainsi le fichier système décrit sur la figure ci-dessous :

image 4

### Élévation des privilèges

Définition
Une élévation de privilèges est réalisable lorsqu'une application ne vérifie pas que l'utilisateur possède les bonnes permissions pour charger telle ressource ou utiliser telle fonctionnalité de l'application.

Un attaquant peu ainsi usurper un autre utilisateur avec les mêmes droits (élévation de privilège horizontale), voire un administrateur (élévation de privilège verticale).
Scénario : Un attaquant force le navigateur à visiter des URL arbitraires
L'application possède une interface dédiée aux administrateurs avec plusieurs fonctionnalités qui leurs sont réservées :
https://example.com/auth/admin/getUsersInfo
Un utilisateur non authentifié ou non-administrateur pouvant accéder à ces pages d'administration caractérise une vulnérabilité d'élévation de privilège. Le serveur ne vérifie pas correctement les privilèges de l'utilisateur et la session associés à la requête.

### protection

Pour se prémunir de ce type de vulnérabilité, il est recommandé de documenter les différents droits, rôles et types d'utilisateurs. À partir, de cette documentation, il est possible de créer une matrice de contrôle d'accès sur les différentes fonctions et ressources de l'application. Celle-ci va permettre de définir à l'écrit qui sont les personnes ayant accès aux différentes ressources. Elle permet ainsi d'éviter les erreurs de développement dans le contrôle des droits et configurer plus facilement des tests unitaires pour la vérification des contrôles d'accès tout le long du développement de l'application.

Voici une liste des points sensibles à porter attention pour éviter la présence d'une vulnérabilité de type Broken Access Control dans une application :

Les identifiants des utilisateurs, rôles, ressources ou fonctions. Beaucoup d'applications utilisent une clef ou un index pour identifier un objet dans une application. Par exemple, l'utilisateur john aura l'identifiant 22, pour accéder à sa facture du mois de juillet, il devra accéder à la ressource avec la clef july_x5fks3z. Lors de l'accès à cette ressource, l'application doit correctement vérifier que cet utilisateur (le numéro 22 ici) ou son rôle lui permet de lire cette ressource. Même si cette clef paraît partiellement aléatoire, la vérification des droits doit se faire systématiquement. Effectivement, un attaquant pourrait réaliser une attaque par force brute, trouver cette clef et y accéder.
Les contrôles d'accès et les mécanismes d'authentification. Certaines applications mettent en œuvre une page de connexion pour accéder à davantage de fonctionnalités. Il est nécessaire de vérifier qu'il soit impossible pour un attaquant de passer au travers et d'accéder à des ressources normalement inaccessibles publiquement.
L'accès à des ressources par leur nom. Des sites utilisent cette méthode pour intégrer dynamiquement des ressources dans une page, par exemple avec paramètre dans une URL : https://www.example.com/?page=news1. Ce paramètre (ici page) doit être vérifié afin qu'un attaquant ne puisse pas entrer le nom et le chemin d'une autre ressource afin d'accéder à un fichier système par exemple. Afin d'éviter ce type d'attaque, il convient de créer une liste blanche des ressources pouvant être chargées par l'application et de bannir les caractères /, \, .. et leurs équivalents encodés.
Les droits systèmes sur les fichiers. Des applications s'appuient sur les droits du système pour le contrôle des accès. Il est donc important de vérifier ceux-ci pour éviter que des ressources soient lues ou modifiées depuis le site web. Par exemple, un fichier de configuration ou un fichier .htaccess peuvent être lus par le serveur ou l'application, mais ne doivent pas être modifiables depuis celui-ci.
La mise en cache côté client. L'accès à un ordinateur peut être partagé comme dans les écoles, bibliothèques, etc. Les développeurs de l'application doivent donc mettre en place des mécanismes pour s'assurer que des informations sensibles ne soient pas mises en cache par le navigateur web.

De plus, l'idéal est de bloquer toutes les ressources par défaut, puis autoriser l'accès à certains rôles ou utilisateurs spécifiquement.

Enfin, les fonctions d'administration peuvent faire l'objet de protections supplémentaires en autorisant leur accès uniquement à partir de certaines IP, d'un navigateur certifié (certificat TLS client) ou d'une connexion VPN.

### référence :

- OWASP Broken Access Control https://owasp.org/index.php/Broken_Access_Control
- OWASP Proactive Controls: Access Controls https://www.owasp.org/index.php/OWASP_Proactive_Controls#6:_Implement_Access_Controls
- OWASP Application Security Verification Standard: V4 Access Control https://www.owasp.org/index.php/ - Category:OWASP_Application_Security_Verification_Standard_Project#tab.3DHome
  OWASP Testing Guide: Authorization Testing https://www.owasp.org/index.php/Testing_for_Authorization
  OWASP Cheat Sheet: Access Control https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html

## CROSS-SITE SCRIPTING (XSS)

Définition
Le Cross-Site Scripting (abrégé XSS) est un type de vulnérabilité de sécurité qui touche les sites Web et permet à un attaquant d'injecter du contenu dans une page, provoquant ainsi des actions sur les navigateurs Web des clients la consultant.

Il existe deux types de XSS :

XSS réfléchie : L'application ne contrôle pas les entrées utilisateur et les restituent sur la page HTML telles que fournies par ce dernier. Une attaque réussie peut permettre à l'attaquant de faire interpréter du code HTML et/ou JavaScript arbitraire dans le navigateur de la victime. Typiquement, la victime devra interagir avec un lien malveillant qui pointe vers une page contrôlée par l'attaquant.

XSS stockée : L'application conserve les données d'entrée non validées et non échappées de l'utilisateur en base de données. Celles-ci sont ensuite restituées dans la page Web pour chaque utilisateur consultant cette dernière. La vulnérabilité XSS stockée est souvent considérée comme un risque élevé ou critique, car elle permet potentiellement de conduire une escalade de privilège si la victime possède plus de droits que l'attaquant. De plus, cette attaque est persistante contrairement à la XSS réfléchie.

Note: Il existe d'autres types de vulnérabilités XSS.

### impact :

Les attaques XSS permettent de réaliser les actions suivantes :

Le vol de session ;
L'usurpation de compte ;
Téléchargement de logiciels malveillants sur l'ordinateur de la victime pour prendre le contrôle de ce dernier ;
Enregistrement des frappes du clavier de la victime ;
Ingénierie sociale : redirection vers une page sous le contrôle de l'attaquant afin de voler des identifiants (Facebook, Gmail, LinkedIn ...) ;
Etc...

### XSS Réfléchie

image 5

Exploitation
Insertion du code malveillant dans l'URL :
image 6

Il est alors possible d'insérer une charge utile malveillante directement dans l'URL (méthode GET utilisée) afin de mettre en évidence la présence de la vulnérabilité XSS.

http://sitevunerable.com/index.php?name=<script>alert('XSS')</script>

Déclenchement de la vulnérabilité XSS réfléchie dans le navigateur :
image 7

### XSS Stockée

image 8
image 9

### Détection

Les outils automatisés peuvent détecter et exploiter toutes les formes de XSS, et il existe des frameworks disponibles gratuitement (OpenVAS, OWASP ZAP).

http://www.openvas.org/
https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project

## DÉSÉRIALISATION NON SÉCURISÉE

Introduction
Les vulnérabilités de type injection d'objet sont provoquées par la mauvaise utilisation des fonctionnalités de sérialisation au sein d'une application.
La sérialisation de données est présente dans de nombreux langages et permet d'avoir une représentation d'un objet sous la forme d'une chaîne de caractères.
Ceci facilite la transmission de données dans l'application ou vers l'utilisateur.

## DÉSÉRIALISATION NON SÉCURISÉE

Introduction
Les vulnérabilités de type injection d'objet sont provoquées par la mauvaise utilisation des fonctionnalités de sérialisation au sein d'une application.
La sérialisation de données est présente dans de nombreux langages et permet d'avoir une représentation d'un objet sous la forme d'une chaîne de caractères.
Ceci facilite la transmission de données dans l'application ou vers l'utilisateur.

La sérialisation d'un objet en PHP est effectuée avec la fonction serialize(). Vous pouvez ainsi stocker le contenu d'une classe dans une base de données par exemple :

<?php

class User {
    public $id;
    public $name;
    public $email;

    function getName(){
        return $this->id;
    }

    function getEmail(){
        return $this->name;
    }

    function getId(){
        return $this->id;
    }
}

$class = new User();
$class->id = 1;
$class->name = "James Jones";
$class->email = "j.jones@mail.com";

print(serialize($class));

?>

L'exécution du code ci-dessus retourne la valeur sérialisée de User, ce qui donne la chaîne de caractères suivante :
O:4:"User":3:{s:2:"id";i:1;s:4:"name";s:11:"James Jones";s:5:"email";s:16:"j.jones@mail.com";}

Maintenant, si nous mettons en entrée de la fonction unserialize() cette chaîne de caractères, nous récupérerons l'objet avec toutes ses propriétés remplies.

Il faut cependant noter que la fonction unserialize() est uniquement capable d'instancier des classes ayant déjà été définies avant son exécution.

Prenons par exemple le code PHP suivant :

<?php 

require('logger.php'); 
require('user.php'); 

$logger = new Logger(); 
$user = unserialize($_GET['user']); 
$logger->addLogEntry('User '. $user->name .' has logged in.'); 

?>

Le fichier logger.php contenant la définition de la classe Logger suivante :

<?php 

class Logger { 
    private $logFile = 'ca-ts.access.log'; 
    private $logData; 
    
    function addLogEntry($string) { 
        $this->logData .= $string . '\n'; 
    } 
    
    function __destruct() { 
        $f = fopen("logs/" . $this->logFile, "a+"); 
        fwrite($f, $this->logData); 
        fclose($f); 
    } 
} 

?>

Cette classe stocke les événements journalisés dans une variable nommée $logData, puis les écrit dans un fichier $logFile au moment de la destruction de la classe grâce à la méthode magique \_\_destruct().

Il est donc possible, à partir de la classe Logger, de créer un exploit très simple permettant d'enregistrer un évènement journalisé sous un autre nom et contenant un code PHP arbitraire. Pour ce faire, nous recréons une classe possédant exactement le même nom (Logger) sur notre ordinateur local et réutilisant les variables $logFile et $logData:

<?php 

class Logger { 
    private $logFile = "exploit.php"; 
    private $logData = "<?php phpinfo(); ?>";

}

print("Serialized Logger : " . serialize(new Logger) . "\n");

?>

Nous obtenons donc le résultat suivant :
:6:"Logger":2:{s:15:"LoggerlogFile";s:11:"exploit.php";s:15:"LoggerlogData";s:19:"<?php phpinfo(); ?>";}

Il ne nous reste plus qu'à appeler le script PHP en mettant ce résultat dans le paramètre user, pour que la fonction Logger->**destruct() exécute le code suivant :
function **destruct() {
$f = fopen("logs/" . "exploit.php", "a+"); 
    fwrite($f, "<?php phpinfo(); ?>");
fclose($f);
}

Il existe un outil automatisé pour exploiter une désérialisation non sécurisée. Cet outil implémente plusieurs chaînes d'exploitation selon les différents cadriciels existants : PHPGCC (https://github.com/ambionics/phpggc).

La sérialisation et désérialisation non sécurisée est aussi présente dans d'autres langages, principalement Java et .NET.
Java
La sérialisation en Java est possible en implémentant l'interface Serializable, puis en utilisant les classes ObjectInputStream et ObjectOutputStream.

Un flux binaire représentant des données sérialisées Java commencera toujours par les deux octets suivants : AC ED (ou 10101100 11101101 en binaire).

Il existe également un outil pour automatiser l'exploitation de désérialisation non sécurisée en Java : ysoSerial (https://github.com/frohoff/ysoserial).
C# / .NET
En environnement .NET, les formats de sérialisations sont basés sur du JSON, XML ou binaire.

Tout comme Java, il existe un outil automatisé pour exploiter une désérialisation non contrôlée : ysoSerial.net (https://github.com/pwntester/ysoserial.net).

Il est vivement recommandé de ne pas utiliser la serialisation d'objet afin de se prémunir de ce type de vulnérabilité. En cas de nécessité absolue, vous pouvez utiliser les solutions suivantes :

En PHP, préférez l'utilisation des fonctions json_encode et json_decode lors de la communication d'objet avec un utilisateur;

Pour Java, en fonction de la bibliothèque utilisée, vous avez la possibilité de désactiver le traitement des entités. Nous vous conseillons de suivre le guide OWASP - XML External Entity Prevention - Java afin d'appliquer la correction adéquate.

Implémentez un système de signature (HMAC) qui peut être couplé avec une solution de chiffrement afin d'empêcher l'altération des données par l'utilisateur.

OWASP Cheat Sheet on deserialization: https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html

Understanding the A8 risk - Unsecured deserialization: https://owasp.org/www-project-top-ten/2017/A8_2017-Insecure_Deserialization

OWASP - XML External Entity Prevention : https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html

## UTILISATION DE COMPOSANTS TIERS AVEC DES VULNÉRABILITÉS CONNUES

### Impacts

Les impacts relatifs aux applications vulnérables sont multiples et de criticité diverse. On peut citer les différents types de vulnérabilités :

Déni de service
Exécution de code arbitraire
Débordement de tampon
Corruption mémoire
Injection SQL, NoSQL, LDAP , etc.
Cross-Site Scripting (XSS)
Fuites d'information technique
Cross-Site-Request-Forgery
Inclusion de fichier local / distant
Etc.

### Composant vulnérable

Système de gestion de contenu Liferay
Nous prenons pour exemple en cible un site internet utilisant Liferay en version 7.2.0-ga1. Cette version de Liferay, souffre d'une vulnérabilité permettant d'exécuter du code Java arbitraire, car non mis à jour malgré les bulletins de sécurité émis par l'éditeur.

https://portal.liferay.dev/learn/security/known-vulnerabilities

Plusieurs codes d'exploitation existent pour cette vulnérabilité:

https://www.synacktiv.com/publications/how-to-exploit-liferay-cve-2020-7961-quick-journey-to-poc.html
https://github.com/mzer0one/CVE-2020-7961-POC

Un attaquant ou même un ver informatique pourrait ainsi facilement prendre le contrôle du serveur à l'aide de scripts d'exploitation disponibles au public.

### Détection

Afin de maintenir à jour son application, il est essentiel de dresser un inventaire des applications, bibliothèques et des différents actifs liés : système d’exploitation, système de gestion de base de données, serveur applicatif, etc.

Pour chaque actif il convient de vérifier que ce dernier est toujours supporté par son éditeur et anticiper sa mise à niveau avant que celui-ci ne devienne obsolète.

Il faut, pour chaque éditeur, s'inscrire à sa liste de diffusion pour être averti des correctifs de sécurité.

Certains outils comme OWASP Dependency-Check (https://owasp.org/www-project-dependency-check/) permettent de vérifier si les dépendances d'un projet Java contiennent des vulnérabilités connues.

### Protection

Afin de prévenir au mieux la survenance d'une vulnérabilité, un processus de déploiement de correctifs sur banc d'essai, puis en production, doit être rédigé.

Il convient de respecter un délai relativement court pour le déploiement de correctifs de sécurité liés aux vulnérabilités les plus critiques.

Afin de limiter les risques, il faut appliquer les résolutions suivantes :

Supprimer les dépendances inutilisées, les fonctionnalités inutiles, les composants, les fichiers et documents d'installation ;
Inventorier en permanence les versions des composants côté client et côté serveur ;
Surveiller en permanence les sources comme CVE et NVD pour détecter les vulnérabilités dans les composants. Utiliser des outils d'analyse de la composition des logiciels pour automatiser le processus ;
S'abonner aux alertes par courriel pour connaître les vulnérabilités de sécurité liées aux composants que vous utilisez ;
Obtenir les composants qu'auprès de sources officielles via des liens sécurisés. Préférer les paquets signés pour réduire la possibilité d'inclure un composant malveillant modifié ;
Surveiller les bibliothèques et les composants qui ne sont pas maintenus ou qui ne créent pas de correctifs de sécurité pour les anciennes versions.

Chaque organisation doit s'assurer qu'il existe un plan continu de surveillance, d'inventaire et d'application des mises à jour ou des changements de configuration pendant toute la durée de vie de l'application.

### référence

OWASP - Using components with known vulnerabilities: https://owasp.org/www-project-top-ten/2017/A9_2017-Using_Components_with_Known_Vulnerabilities
OWASP - Project Application Security Verification Standard: https://owasp.org/www-project-application-security-verification-standard/
OWASP - Dependency-Check: https://owasp.org/www-project-dependency-check/

## BROKEN AUTHENTICATION

### Définition

Une vulnérabilité de type "Broken Authentication" implique une faiblesse dans le processus d'authentification d'une application.

Cette fonctionnalité est généralement le point d'entrée des clients et des administrateurs pour accéder à leurs droits.

### Causes

Une vulnérabilité "Broken Authentication" peut survenir à cause des défauts suivants :

La possibilité d'automatiser sans restriction l'authentification, permettant notamment à un attaquant de tester rapidement des couples identifiants et mots de passe pour en trouver un valide (brute force ou attaque par dictionnaire).
L'application indique si un nom d'utilisateur est associé à un compte valide.
La modification des mots de passe par défaut n'est pas forcée.
Une politique de mots de passe robuste n'est pas mise en place et son application n'est pas forcée.
Le stockage et la transmission des mots de passe ne sont pas sécurisés.
Les jetons d'utilisateur ne sont pas correctement invalidés après une déconnexion.
Les cookies de session ne sont pas aléatoires et peuvent être devinés par un attaquant.
Etc...

### Exemple

Une vulnérabilité de type "Broken Authentication" peut impliquer la possibilité d'automatiser le processus de connexion.

Si aucune limitation n'est mise en place, l'attaquant va ainsi pouvoir automatiser et tester un énorme nombre de combinaisons jusqu'à ce que l'une d'entre elles soit valide.

Néanmoins, il faudra noter que le taux de réussite de ce genre de tentatives est faible, car généralement ces attaques se font avec des identifiants et mots de passe aléatoires.

Cependant, d'autres faiblesses, généralement appelées fuites d'information, peuvent faciliter ce type d'attaque pour cibler plus précisément des utilisateurs.
Page vulnérable
L'authentification sur cette page peut être automatisée par force brute :

Il existe de nombreux logiciels permettant ce type d'attaques :

Wfuzz (https://github.com/xmendez/wfuzz)
Hydra (https://github.com/vanhauser-thc/thc-hydra)
Patator (https://github.com/lanjelot/patator)
Etc...

### Protections

Afin de protéger son application et les utilisateurs, plusieurs mécanismes peuvent être mis en place :

Mettre en place un mécanisme de double authentification pour les applications les plus sensibles, permettant de se protéger contre les attaques automatisées, dont la réutilisation de mot de passe.
Forcer la modification d'un mot de passe par défaut dès l'installation du logiciel et avant qu'il soit accessible sur un autre réseau comme Internet.
Implémenter une politique de mots de passe robuste, avec au minimum :
12 caractères alphanumériques ;
au moins une majuscule ;
au moins une minuscule ;
au moins un caractère spécial ;
ne possède aucun lien avec l’entité protégée ;
n'est pas présent dans les mots de passe les plus mauvais couramment utilisés ;  
n'est pas un ancien mot de passe réutilisé.
Forcer le changement régulier des mots de passe des utilisateurs et des administrateurs, environ tous les 90 jours pour les applications sensibles.
Envoyer des messages génériques en cas d'erreur ne permettant pas d'identifier si la connexion est effectuée avec un nom d'utilisateur valide.
Enregistrer toutes les tentatives d'authentification échouées pour détecter une potentielle attaque
Limiter le nombre de tentatives de connexion par adresse IP, voire mettre en place un Captcha.
Enregistrer les échecs de connexion et prévenir l'utilisateur ou l'administrateur d'un nombre de tentatives de connexion infructueuses élevé.
Définir un temps d'expiration faible après inactivité de l'utilisateur.
Transmettre les mots de passe de manière chiffrée au serveur (par exemple une couche TLS).
Stocker les mots de passe de manière sécurisée sur le serveur à l'aide d'une fonction de hachage robuste.
Bloquer le compte pour une durée déterminée après plusieurs tentatives échouées de connexion.

### Références

OWASP - A2:2017-Broken Authentication (https://owasp.org/www-project-top-ten/2017/A2_2017-Broken_Authentication.html)
OWASP - Authentication Cheat Sheet (https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
auth0 - What Is Broken Authentication? (https://auth0.com/blog/what-is-broken-authentication/)

## SENSITIVE DATA EXPOSURE

### Définition

Cette vulnérabilité indique que des données sensibles sont exposées de manière non sécurisée.
En effet, ces dernières seraient envoyées en clairs dans le réseau ou stockées, toujours en clairs, ou chiffrées/hashées avec un algorithme faible sur le serveur sans restriction d'accès.
Ainsi le serveur commet un manquement grave à la sécurité de ces données, d'autant plus qu'il peut s'agir de mots de passes, numéros de cartes de crédit, informations sensibles sur une société, etc.

Les points amenant à cette vulnérabilité sont :

Utilisation des protocoles non sécurisés: HTTP, SMTP et FTP
Données sensibles stockées en clair
Clés de chiffrement compromises/faibles/anciennes
Algorithmes de chiffrement faibles/anciens
Algorithmes de hashage faibles/anciens
Mauvaise gestion des accès aux données

### Impacts

Les impacts sont critiques car ces données sont dites sensibles, comme les exemples cités plus haut.

Dans un réseau local, l'attaquant pourra effectuer un man-in-the-middle entre la victime et le serveur et ainsi récupérer les identifiants et mots de passe de session.

Sinon, dans le cas d'un réseau wifi ouvert, il aura la possibilité de subtiliser le cookie de session d'un utilisateur.
Fonctions de hashage
Les fonctions de hashage sont des fonctions à sens unique.
A partir d'une même entrée, elles génèrent toujours le même
condensat (de longueur fixe).

Ces fonctions sont conçues pour ne pas pouvoir retrouver les données d'entrée à partir du condensat.

Elles sont très utiles en cryptographie et pour le stockage des mots de passe.

Elles permettent de ne stocker que le condensat, jamais le mot de passe de l'utilisateur.

Un seul doit être utilisé pour cette application, car deux utilisateurs
ayant le même mot de passe auraient le même condensat dans la base de données.

## XML EXTERNAL ENTITIES

### Définition

Par défaut, de nombreux parseurs XML autorisent la définition d'entités externes.
Celles-ci se présentent sous la forme d'une URI qui sera déréférencée puis évaluée lors du traitement XML.

Un attaquant est en mesure d'exploiter ce type de vulnérabilité s'il est capable de téléverser des fichiers XML arbitraires, ou alors d’interagir avec le contenu d’un document XML.
Le risque étant que l’application traite et/ou exécute l’injection malveillante.

### Impacts

Une vulnérabilité XXE peut provoquer les impacts suivants :

Lecture de fichiers arbitraires locaux
Accès à des services locaux
Accès à des services distants
Déni de service
Dans un cas particulier (PHP avec le module expect installé) : exécution de commandes arbitraires

## Exemple

Nous pouvons ici prendre l'exemple d'une API SOAP. Ce type d'API est basé sur un protocole XML, nous présentant donc une cible facile.
Nous avons ici un service SOAP permettant de récupérer les données d'un pays :

POST /ws HTTP/1.1

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:gs="http://example.com">
<soapenv:Header/>
<soapenv:Body>
<gs:getCountryRequest> : Fonction proposée par l'API
<gs:name>Spain</gs:name> : Paramètre de la fonction
</gs:getCountryRequest>
</soapenv:Body>
</soapenv:Envelope>

Ayant accès à la requête transmise il nous est possible de la modifier afin d’injecter du code malveillant

===================

HTTP/1.1 200

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
<SOAP-ENV:Header/>
<SOAP-ENV:Body>
<ns2:getCountryResponse xmlns:ns2="http://example.com">
<ns2:country>
<ns2:name>Spain</ns2:name>
<ns2:population>46704314</ns2:population>
<ns2:capital>Madrid</ns2:capital>
<ns2:currency>EUR</ns2:currency>
</ns2:country>
</ns2:getCountryResponse>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>

Comme nous soumettons du XML arbitraire au serveur, nous pouvons essayer de définir une entité externe à l'aide d'un DTD (Doctype definition) :

<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >

]>
Ce DTD définit une entité xxe qui sera substituée au contenu du fichier /etc/passwd à chaque endroit où elle sera mentionnée.

Le fichier /etc/passwd contient pour les systèmes linux toutes les informations relatives aux utilisateurs du système (login, invite de commande, etc.).

Nous pouvons donc changer notre requête SOAP comme ceci :

<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >

]>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:gs="http://spring.io/guides/gs-producing-web-service">
<soapenv:Header/>
<soapenv:Body>
<gs:getCountryRequest>
<gs:name>&xxe;</gs:name>
</gs:getCountryRequest>
<!-- On intègre la variable contenant le contenu du fichier dans le but que le serveur nous affiche le rendu -->
</soapenv:Body>
</soapenv:Envelope>
Ce qui résultera en la réponse suivante de l'API :
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
<SOAP-ENV:Header/>
<SOAP-ENV:Body>
<ns2:getCountryResponse xmlns:ns2="http://example.com">
<ns2:country>
<ns2:name>root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
[...ETC...]
</ns2:name>
</ns2:country>
</ns2:getCountryResponse>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
On voit donc ici que le serveur a exécuté le code malveillant que nous avons intégré, nous révélant des informations que nous n’aurions pas dû obtenir normalement.

### Prévention

Lorsqu'une application doit traiter du contenu XML n'étant pas de confiance, il faut désactiver la définition de type de document.

Pour cela, l'OWASP dispose d'une ressource exhaustive :

XML External Entity Prevention Cheat Sheet (https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)

### Références

OWASP - XML External Entity Prevention Cheat Sheet(https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
OWASP - XML External Entity (XXE) Processing (https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)

## SECURITY MISCONFIGURATION

### Définition

Une mauvaise configuration de sécurité est le problème de sécurité rencontré le plus courant.
Ceci est généralement le résultat de configurations par défaut non sécurisées ou incomplètes, d'un stockage dans le cloud ouvert, d'en-têtes HTTP mal configurés et de messages d'erreur détaillés contenant des informations sensibles.
Non seulement tous les systèmes d'exploitation, frameworks, bibliothèques et applications doivent être configurés en toute sécurité, mais ils doivent également être corrigés / mis à jour en temps opportun.

Voici d'autres exemples de mauvaises configurations :

Permissions d'accès mal configurées, ou inexistantes
Fonctionnalités activées, mais non nécessaires
Mot de passe par défaut
Le logiciel utilisé est déprécié ou vulnérable
Les paramètres de sécurité dans les serveurs d'applications, les frameworks d'application (par exemple Struts, Spring, ASP.NET ou Symfony), les bibliothèques, les bases de données, etc. ne sont pas définis sur des valeurs sécurisées.

### Exemples

Symfony
Symfony est un framework PHP, très répandu dans l'industrie.

Jusqu'à sa version 3.4.44, le secret utilisé pour signer certains jetons était par défaut ThisTokenIsNotSoSecretChangeIt.
Lors de son installation, il n'était pas changé à une valeur sécurisée aléatoire et c'était le rôle de la personne exécutant l'installation.

Si la valeur de ce secret n'était pas changée, il devenait alors possible d'exécuter du code arbitraire à distance.
(Pour plus d'informations, vous pouvez consulter le blog du laboratoire d'Ambionics (https://www.ambionics.io/blog/symfony-secret-fragment) sur ce sujet).
Firebase
Firebase est une base de données en temps réel proposée par Google.
Son intérêt principal est la synchronisation de données applicatives dans le cloud.
Elle est majoritairement utilisée dans un contexte mobile (Android, iOS).

Une base de données Firebase se présente sous la forme d'une API REST, dont l'URL est : https://<ID_DU_PROJET>.firebaseio.com.
Un moyen efficace de trouver l'URL exacte est de réaliser une décompilation d'une application mobile.

Le problème est présent (et courant !) lorsque l'administrateur de cette base de données autorise l'accès public en lecture, voire plus grave : l'accès public en écriture.

Il devient ainsi possible d'extraire l'ensemble de cette base de données, simplement en ajoutant .json à la fin de l'URL (exemple : https://mon-projet-e532.firebaseio.com/.json), ou encore d'insérer du contenu arbitraire dans celle-ci à l'aide d'une requête HTTP PUT :
curl -X PUT https://mon-projet-e532.firebaseio.com/test.json -d '{"data": "test"}''

Sortie de container Docker
Docker est un outil qui peut empaqueter une application et ses dépendances dans un conteneur isolé, qui pourra ensuite être exécuté sur n'importe quel serveur.

Cet outil est très largement utilisé de nos jours, afin de faciliter le déploiement des applications et de mieux gérer les montées en charge.

Par défaut, un conteneur Docker est sécurisé (sous réserve que l'hôte soit mis à jour et ne possède pas de vulnérabilités dans son noyau) et peut être utilisé comme un "bac à sable". Cependant, il est possible de remonter vers l'hôte et d'exécuter des commandes sur celui-ci dans deux cas :

Le conteneur est lancé avec le drapeau --privileged : dans ce cas, un utilisateur root dans le conteneur peut exécuter des commandes en tant que root sur l'hôte (soit en montant le disque de l'hôte dans le conteneur, soit en exploitant la fonctionnalité (https://medium.com/better-programming/escaping-docker-privileged-containers-a7ae7d17f5a1) release_agent de cgroups).
Le socket Docker est partagé dans le conteneur : Le socket Docker est un accès direct et non contrôlé à l'hôte (https://dejandayoff.com/the-danger-of-exposing-docker.sock/), et permet donc d'exécuter des commandes en tant que root sur celui-ci. Ceci est dû au fait que n'importe qui ayant accès au socket Docker peut créer et lancer des containers, et donc utiliser la technique précédente. C'est aussi pour cette raison que sur l'hôte Linux, seul l'utilisateur root et les utilisateurs membres du groupe docker peuvent accéder à ce socket par défaut.

### Protection

Pour éviter ce genre de problème, administrateurs et développeurs devront suivre les guides de bonnes pratiques et être attentifs aux politiques de sécurité :

Suivre ou mettre en place les informations de sécurité de l'éditeur lors d'un processus d'installation
Supprimer les fonctionnalités non nécessaires et limiter l'accès aux autres
Mettre à jour régulièrement les composants de l'application
Structurer les applications et mettre en place des barrières d'accès entre elles
Spécifier les directives de sécurité au navigateur client

### détections

Il existe des outils répertoriant nombre de ces défauts de configuration et permettant de les identifier :

Nikto (https://github.com/sullo/nikto)
Acunetix (https://www.acunetix.com/)
Skipfish (https://github.com/spinkham/skipfish)
Etc...

Alternativement, chacune de ces vulnérabilités peut être testée manuellement sur le site Web avec plus de minutie.
Des tests plus poussés pourront être aussi effectués.

### Références

OWASP - Security Misconfiguration (https://owasp.org/www-project-top-ten/2017/A6_2017-Security_Misconfiguration)
OWASP Testing Guide - Configuration and Deployment Management Testing (https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/README)

## JOURNALISATION ET MONITORING DÉFICIENT

### Définition

Une politique de journalisation insuffisante des évènements survenant aux niveaux applicatifs, réseaux et systèmes permet de fournir à un attaquant plusieurs opportunités de sondage et d'exploitation des actifs sans laisser de traces.

Une architecture typique de journalisation va tenter de générer, stocker, analyser et surveiller les journaux de manière sécurisée.

Il est important ici de différencier les journaux opérationnels et les journaux de sécurité :

Les journaux opérationnels vont inclure les évènements de routine du système tel que l'identification, les erreurs jetées par le système par exemple. Ils vont permettre de retracer le parcours d'un utilisateur.
Les journaux de sécurité, quant à eux, sont produits directement par les équipements de sécurité et vont typiquement inclure des alertes sur des attaques ou anomalies potentielles en cours.

Dans tous les cas, ces deux types de journalisation sont indispensables et ne doivent pas être sous-estimés car ils permettent, lors de leur bonne implémentation, d'anticiper une attaque ou de reconstruire la chronologie d'une attaque passée.

Quelques exemples de vulnérabilité de journalisation peuvent être :

Des évènements ne sont pas journalisés (identification non réussie par exemple);
Les journaux sont uniquement stockés sur le système local et ne sont pas sauvegardés sur un autre système;
Les alertes sont ignorées ou les activités malicieuses ne sont pas détectées en temps réel.

### Scénarios d'attaque

- Un logiciel open-source, géré par une petite équipe, a été piraté en utilisant une faille contenue dans ce logiciel. Les attaquants ont réussi à supprimer le dépôt de code interne contenant les développements en cours. Même si une partie du code a pu être récupérée (via les copies locales des développeurs), l'absence de journalisations, surveillances et alertes a rendu l'impact de cette attaque plus important.

- Un attaquant tente de se connecter sur plusieurs comptes utilisateur avec un mot de passe commun ("Password Spraying"). Quelques comptes se feront pirater, mais la plupart des autres comptes n'auront qu'une seule tentative d'authentification échouée. Une solution de surveillance aurait pu corréler les journaux d'authentification et alerter de l'attaque en cours.

- Une entreprise majeure possède un antivirus analysant les pièces jointes des emails. Cet antivirus a remonté un avertissement concernant un logiciel potentiellement malicieux, ces avertissements ont continué a être produits pendant un certain temps avant que l'attaque ne soit finalement détectée, trop tard.

### Prévention

- S'assurer que tous les évènements d'authentification, contrôle d'accès et validation d'entrée utilisateur générant une erreur soient journalisés avec suffisamment de contexte pour identifier les activités suspicieuses ou malicieuses. Ces journaux doivent être conservés suffisamment de temps pour que des investigations numériques restent possibles à posteriori.

- Les journaux doivent être générés dans un format qui peut facilement être ingéré par les solutions de centralisation de journaux.

- Les transactions sensibles doivent avoir une trace d'audit ayant un contrôle d'intégrité (ajout dans une base de données en lecture seule par exemple). Ceci pour éviter qu'un attaquant puisse supprimer ou modifier ces journaux.

- Assurer la mise en place d'une solution de surveillance et d'alerte afin que les activités suspicieuses soient détectées et adressées le plus rapidement possible

### Références

OWASP Insufficient Logging & Monitoring (https://owasp.org/www-project-top-ten/2017/A10_2017-Insufficient_Logging%2526Monitoring)- [OWASP Implement Security Logging and Monitoring](https://owasp.org/www-project-proactive-controls/v3/en/c9-security-logging.html)
OWASP Logging Cheat Sheet (https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
