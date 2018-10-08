

### Mots Clés
-   IPC: communication inter-processus, ensemble de mécanisme permettant à des processus concurrents de communiquer ça peut être pour : l'échange de données, la synchronisation entre processus et l'échange de donnée ET la synchronisation entre processus 
-   Fichier de Log: historique, fichier gardant trace de tous les événements affectant un processus
-   RPC: remote procedure call, protocol permettant la comm en tre processus 
-   NFS / DES: Network File System et Distriuted file system, protocoles permettant le partage de fichier sur un réseau de façon transparente à l'utilisateur
-   Socket:  addresse:port
-   Couche IP
-   Système de fichier
-   Get / Post
-   Fichier sur un réseau
-   Gestion des droits d’accès
-   Protocole
-   Communication inter-processus
-   Conteneurs
-   Cloud


### Contexte
Quoi ?
-   Exploiter un système de fichier distant comme s’il était local
Comment ?
-   En étant basé sur RPC
-   En passant par le cloud
Pourquoi ?
-   Pour faciliter l’accès aux fichiers
-   Pour gérer les droits d’accès
Contraintes
-   Rien
Problématique
-   Comment créer un système de fichier distribué ?
Généralisation
-   Partage d’information
Hypothèses
-   Un système de fichiers distribués permet d’interconnecter plusieurs périphériques de stockage pour créer qu’une “seule grande masse”
-   Le rpc est basé sur des talons.
-   Le système de fichiers distribués permet de cumuler le stockage de plusieurs zones géographiques.
-   NFS est un protocole de partage de fichier.
-   NFS est un serveur de partage de fichier, DFS est le protocole utilisé pour centraliser les accès.
-   Tous les systèmes de fichiers distribués ne gèrent pas les droits d’accès
-   Certains systèmes de fichiers distribués ont besoin d’un orchestrateur pour fonctionner.
-   On peut gérer les droits grâce aux sockets.

### Plan d’Action
### Études
####  Systèmes de fichiers distribués + Lister les principaux

Un système de fichier distibué ou système de fichier en réseau est un système de fichier 
permettant le partage de fichiers entre plusieurs machines/clients à l'aide d'un réseau informatique. Contrairement à un sytème de fichier local, le client n'a pas accès au système de stockage sous-jacent, et interagit avec le système de fichiers via un protocole.

liste des principaux :
-   AFS - Andrew File System
-   CephFS
-   Coda
-   GlusterFS
-   GFS - Google File System
-   Hadoop Distributed File System (HDFS)
-   InterMezzo (système de fichiers)
-   Lustre
-   MooseFS
-   NFS - Network File System
-   RozoFS
-   SheepDog
-   Unity, du logiciel Perfect Dark
-   Tahoe-LAFS
-   XtreemFS

il spécifie:
modèle et achi du fichier : décrit le fonctionnement et les ressources
méthode d'accès au ressources : comment l'user peut accéder les fichiers
les opérations disponibles
un protocole de message qui gère le format des données échangées
outils administratifs

####   RPC

différentes implémentation de RPC :

-   Sun ONC/RPC
-   TI-RPC
-   OSF DCE open soft foundation distrib comp env, framework pour comm client/serv
-   OMG CORBA: object managment group common object request borker architecture
-   Sun Java RMI: 
-   Sun J2EE EJB
-   WS-SOAP

**Remote Procedure Call** : idée émanant du concept d'appel de fonction, l'idée est de programmer des applications distribuées en appelant des fonctions qui sont situées sur une machine distante.

On peut l'utiliser pour toute sorte d'applications distribuées. ex: les clients légers

De nombreux système RPC existent et ne sont évidement pas compatible entre eux

Sun RPC devenu un standard puisque ses specs sont dans le domaine public. Il a été dev pour servir de base au système NFS de Sun qui est largement utilisé sous Linux

RPC essaye de maintenir le plus possible la sémantique habituelle d'appel de fontions. Il y a dans le programme client une fonction locale ayant le même nom que la fonction distante qui appelle d'autres fonctions de la bibliothèque RPC qui prennent en charge les connexions réseaux, le passage des paramètres et le retour des résultats.

Côté serveur il suffit d'écrire une fonction comme en dev et un processus se charge d'attendre les connexions clientes, appeler la fonction avec les bons paramètres et se chargera ensuite de renvyer les résultats. Ces fonctions sont les "stubs", il faut donc écrire un stub client et un stub serveur en plus du programme client et de la fonction distante.

Le travail nécessaire à la construction des stubs client et serveur sera automatisé grâce au programme rpcgen qui produira du code C qu'il suffira alors de compiler. Il ne restera plus qu'à écrire le programme client, qui appelle la fonction et en utilise le résultat (par exemple en l'affichant), et la fonction elle-même.

Le système RPC n'autorise qu'un seul argument en paramètre et un seul en retour. Pour passer plusieurs arguments on utilise donc une structure. De même, pour renvoyer plusieurs valeurs.

#### Écrire un programme RPC
Il y a en fait deux programmes, un programme client et un programme serveur. Le travail commence donc par la définition de l'interface en utilisant l'IDL (Interface Definition Language) du système RPC (proche du C).

Ce fichier de définition est ensuite traité par l'utilitaire rpcgen (RPC program generator). Il suffit de taper sur la ligne de commande :

rpcgen -a calcul.x

-a permet de produire un squelette pour le programme client et un pour la fonction distance
la commande produit un fichier d'en-tête programme.h, un stub serveur programme_svc.c, un stub client programme_clnt.c et des routines XDR programme_xdr.c

XDR (eXternal Data Representation) définit les types utilisés pour l'échange de variables entre client et serveur. Les deux prcessus peuven ne pas tourner sur la même plateforme et doivent donc utiliser un langage commun

**processus serveur**

on peut cosulter le squelette de fonction avec rpcgen -a

il n'y a pas de fonction main dans prgramme_server.c, elle est située sur le stub du serveur, il s'occupe de recevoir et dispatcher les appels aux fonctions adéquates. 

pour obtenir le programme serveur complet il faut lier calcul_svc.o, calcul_server.o et calcul_xdr.o ensemble

gcc -o server calcul_svc.o calcul_server.o calcul_xdr.o

On peut alors démarrer le serveur, puis utiliser rpcinfo pour vérifier qu'il tourne :
```
$ ./server &
[1] 2746

$ rpcinfo -p
   program no_version protocole  no_port
    100000    2   tcp    111  portmapper
    100000    2   udp    111  portmapper
 536870913    1   udp    803
 536870913    1   tcp    805

$ rpcinfo -u localhost 536870913
Le programme 536870913 de version 1 est prêt et en attente.
```
-p permet de connaître la liste des programmes RPC actuellement enregistrés sur la machine. chaque prog a un numér de version, le protocole et le port utilisé
-u permet de tester le programme indiqué en appelant sa procédure 0

**processus client**

on utilise le squelette fourni par rpcgen : il comprend un appel à chacune des fonctions définies dans l'interface et chaque appel est suivi d'un test qui détecte les erreurs de niveau « RPC » (serveur ne répondant pas, machine inexistante).

L'erreur éventuelle est alors explicitée par la fonction clnt_perror().

il faut également compiler programme_clnt.o et programme_xdr.o
```
gcc -c programme_client.c
gcc -o client programme_client.o programme_clnt.o programme_xdr.o
```

**résumé**
Avec l'aide de la bibliothèque de fonctions RPC (clnt_create, clnt_destroy ...) et des outils comme rpcgen et rpcinfo, il a été très facile de construire une application simple mais toutefois représentative du mécanisme RPC.
note : on peut noter que RCP se fait progressivment remplacer par CORBA(Common Object Request Broker Architecture) où il n'est plus questions de fonctions distantes mais d'objets distants

**stub client** module d'une application client contennant toutes les fonctions nécessaires pour que le client appelle les procédures à distance. Il invoke le moteur de marshalling et certains API RPC
marshalling: transformer la représentation mémoire d'un objet en format de donnée pour stockage ou transmission

**stub server** module d'une application ou service client(e) contenant toutes les fonctions nécessaires pour que le serveur prenne en charge les requêtes à distance en utilisant des appels de procédure locales

exemples de services Windows server 2003 dépendant de RPC system service (RPCSS) (juste les plus importants:

- Background Intelligent Transfer Service
- DHCP Server
- DNS Server
- File Replication Service
- Kerberos Key Distribution Center
- Logical Disk Manager
- Messenger
- Remote Registry
- Removable Storage
- Routing and Remote Access
- Task Scheduler
- Telnet
- Virtual Disk Service
- Windows Installer
- 


####  NFS / DFS (OSI, TCP/IP, ...)

Network File System permet aux utilisateurs de partager des fichiers de façon transparente
NFS a commencé à permettre aux utilisateurs de partager des fihciers par réseau en utilisant UNIX

Il fût créé par Sun Microsystems pour être plus proche de la gestion de fichier tel qu'elle se fait en local contrairement au fonctionnement de TCP et Telnet qui existait à l'époque. Le but était que pour l'tilisateur, un fois le système configuré l'accès au fichier distant puisse se faire comme si il était sur le disque local

avec NFS  on peut config un disque ou dossier pour être partagé, les clients peuvent ensuite *"monter"* le dossier/disque paratagé
l'architecture NFS inclue 3 composants principaux : XDR, RPCet Mount Protocol
les des buts principaux de NFS était la performance ,il utlise donc UDP, et la simplicité, il est donc stateless (d'où UDP) , cela permet aux requêtes d'être faites indépendement les unes des autres et laisse le serveur gérer les crashes.
Note : le protocol est également fait pour que si la requête est perdu ou dupliquée il n'y a pas de corruption de fichier

NFS v2 a été la première fortement utilisé et est encore la plus utilisée, codifié en standard officiel TCP/IP RFC 1094 en 1989
NFS 3 a été dev en 1995 RFC 1813, similaire à la v2 avec des tailles de fichiers plus grands et plus fa façons d'acéder aux ressources
NFS 4 publié en 2000 RFC 3010 puis en 2003 RFC 3530 qui est une réécriture de NFS:
-  plus de sécurité
- introduit le concept de procédure composée (compond procedure) permettant l'envoie de plusieurs procédures siples de client à serveur
- double du nombre de procédures individuelles que le client peut utiliser pour accéder à un fichier
- intégration des fonction du Mount Protocol dans NFS

architecture NFS :

couche 7 (application) de TCP/IP (donc session+présentation+application d'OSI)

![](![](http://www.tcpipguide.com/free/diagrams/nfscomponents.png))
NFS peut être décomposer en 3 compsants correspondant aux trois couches:

- Remote Procedure Call (RPC): service de la couche session 
- Extarnal Data Representation (XDR) pour la couche présentation qui permet de définir des types de donnée pour les échanger avec NFS entre orid utilisant différente méthodes de stockage interne
- les procédures et opérations NFS : spécifient les tâches à exécuter sur les fichiers et utilise comme on l'a dit XDR pour représenter les données et RPC pour envoyer les commandes à travers le réseau

en plus de ces 3 composants NFS possèdes d'autre fonctions: 
- le Mount Protocol que NFS utilise pour monter les dossiers et qui est inclu dans NFSv4
- le modèle de système de fichier (file system model) NFS utilisé pour la structure des fichiers et dossiers qui est basé sur celui d'UNIX
- la sécurité: utilisant une authentification à la UNIX dans les v2 et 3 pour vérifier les permisions et l'amélioration des méthode de sécu avec la v4 ajoutant notament des options d'encryption


XDR : RFC 1014 puis 1832 en 1995

| donné | taille | description
|--|--|--|
|int |4 | 
|unsigned int |4 |
|enum |4 |
|bool |4 |
|hyper |8 | long
|unsigned hyper |8 |
|float |4 |
|double |8 |
|quadruple |16 | 
|opaque |variable | donnée à transmettre sans avoir de représentation XDR, c'est une black box
|string |variable |
|array |variable | tableau des autres éléments 
|struct |variable |
|union |variable | c'est compliqué....
|void |0 |
|const |0 |

on peut également créé des types de donné


RPC:
Pour les opérations comme la lecture ou l'écriture de fichiers il est plus efficace pour un programme d'appeller des procédures a.k.a programmes conçus à cet effet. Dans le cas de NFS il faut les appellé sur un pc distant, plutôt qu'intégrer ces appels dans NFS Sun a créé un protocole de couche session pour le faire. Il est maintenant utilisé par d'autres protocoles que NFS 

puisque c'est RPC qui effectue les modif de fichiers, NFS n'est qu'un essemble d'opérations et 	procédures RPC

pour effectuer une action le client utilise RPC pour appeller le serveur NFS, le serv accepte la requpete et performe l'action un code de résultat (result code) et potentiellement des données 
note : NFS utilise le port 2049 mais peut en utiliser d'autres avec le "port mapper" de RPC

Procédures NFS:
chaque procédure NFS représente une action que le client peut effectuer surun fichier lecture/écriture, création/suppression de dossier,... l'opération requiert que le fichier soitt référencé par une structure de donnée appellé le file handle (descriptieur/ poignée? de fichier) 
NFS 3 a enlevé 2 procédures de NFS 2 ,en a ajouté d'autres et a changé les nmbres identifiant chaque procédure


procédures:

|#v2|#v3| nom| description
|--|--|--|--|
|0 |0 |null | |
|1 |1 |getattr |get file attribute |
|2 |2 |setattr | set file changes|
|3 |- |root | récupère la racine (obsolète fait parti du mount protocol) |
|4 |3 |lookup | retourne le file handle du fichier actuel |
|5 |5 |readlink | lis le nom du fichier d'un symlink donnée |
|6 |6 |read | lis le fichier |
|7 |- |writecache | jamais utilisé |
|8 |7 |write | |
|9 |8 |create | |
|10 |12 |remove | |
|11 |14 |rename | |
|12 |15 |link | créé un hard link (non symbolique) |nfs close
|14 |9 |mkdir | |
|15 |13 |rmdir | |
|16 |16 |readdir | |
|17 |-|statfs |donne des infos sur le système de fichier distant (taille du fichier, espace libre) fsstat et fsinfo dans la v3 |
|- |4 |access |détermine les droit d'accès pour un fichier |
|- |11 |mknod |créé un fichier spécial (named pipe ou device fie par ex) |
|- |17 |readdirplus |récup des info supplémentaires sur le dossier |
|- |18 |fsstat | retourne des infos dynamiques sur le status du syst de fichier (espace libre nombre de places libres) |
|- |19 |fsinfo | retourne des infos  statiques sur le syst de fichier données d'utilisation, structure des requêtes|
|- |20 |pathconf | etourne des données supplémentaires sur unfichier ou dossier |
|- |21 |commit |écris le contenu du cache du serv et le vide |


avec NFSv4 on peut faire des procédures composées plutôt que de faire 1 requête pour chaque action. Le serv reçoit la procédure composée et exécute les opération dans l'ordre

liste des oprations 
http://www.tcpipguide.com/free/t_NFSServerProceduresandOperations-3.htm




#### DFS

Distributed File System permet de grouper des fichiers présents sur différents serveurs en les connectant à un ou plusieurs namespace

un namespace est un vue virtuelle (virtual view ) des fichiers partagés dans une organisation

Avec DFS les admins peuvent sélectionner quels dossiers on peut inclure dans un namespace ainsi qu'une hiérarchie .

Du point de vue utilisateur le dossier semble être présent sur un disque et peuvent naviguer dans le namespace sans savoir le nom du serveur ou des fichiers partagés

Avantages:

- migration de données simplifiée: puisque le client n'a pas à conaitre le nom du serveur ou des fichiers on peut déplacer les données sans avoir à reconfigurer  les applications et les racourcis
- disponibilité des fichiers: en cas d'échec d'un serveur DFS redirige le client vers un autre serveur pour garantir l'accès permanant aux fichiers
- load sharing (partage de charge) : DFS peut faire du partage de charge pour les fichiers très utilisés  
- sécurité: la sécurit est géré par NTFS (New Technology File System, système de fichier de Windows) 

Cas d'utilisation:

- consolidation de serveurs
- publication de logiciels
- augmenter la disponibilité des données
- utiliser FRS (File Replication Service) pour garder les fichiers synchronisés entre serveurs
- évolutivité

Dépendances de DFS: 
- réplication d'Active Directory
- SMB Server Message Block protocole utilisé par les clients pour accéder aux serveurs root
- RPC
- activer le service DFS sur tous les serv root et contrôleurs de domaine
 

**Accès à un volume DFS**

Vous accédez à un volume DFS à l'aide d'un nom UNC standard :

\\Nom_de_serveur\Nom_de_partage_DFS\Chemin\Fichier

dans lequel  _Nom_de_serveur_  est le nom de l'ordinateur hôte du DFS,  _Nom_de_partage_DFS_  correspond au partage désigné comme la racine de votre DFS, et  _Chemin\Fichier_  est un nom de chemin Win32 valide


3 trucs imo: la racine (serv) , le dossier (nomdu dosser dfs) et la cible (chemin


####  Sockets (Viteuf)

end point d'une communication, combinaison addresse:port permettant d'identifier où les segments doivent être envoyés. Ex: 192.168.0.1:80 précise a quel machine envoyer les informations et quels port (= processus) les réucpère

### Réalisations
#### Corbeille

