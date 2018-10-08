
### Mots Clés
-   IPC* : Inter-Process Communication - Mécanismes permettant à des processus concurrents de communiquer :  d'échange data entre PS, sync entre PS.
-   Fichier de Log*
-   RPC* : Remote Procedure Call
-   NFS / DES* : / 
-   Socket* : @IP + n° port 
-   Couche IP
-   Système de fichier
-   Get / Post
-   Fichier sur un réseau
-   Gestion des droits d’accès
-   Protocole
-   Communication inter-processus* : IPC
-   Conteneurs
-   Cloud
(* à définir)

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
-   Étudier
-   Systèmes de fichiers distribués + Lister les principaux
-   RPC
-   NFS / DFS (OSI, TCP/IP, ...)
-   Sockets (Viteuf)
-   Réalisations
-   LACORBEILLED’EXERCICES


## 1 - Systèmes de fichiers distribués :

- Système qui permet de partage de fichiers à plusieurs clients au travers du réseau informatique.
- Le client n'a pas accès au stockage sous-jacent.
- Utilise un protocole adéquat 
- Ex :
	- NFS
	- AFS (inspiré du NFS)
	- Ceph (données répliquées, tolérant aux pannes)
	- Coda (inspiré NFS, informatique mobile)
	- {...}

Un système de fichier se compose :
- Modèle / archi fichier
- Méthodes accès ressources
- Méthodes opérations pour manipuler le fichier
- Outils administratifs

## 2 - Le RPC :

- Remote procedure Call
- Appeler des fonctions qui sont situées sur une machine distante.
- Traiter des calculs (pratique pour centraliser sur un ordi puissant)
- Microsoft RPC 
- "SUN RPC" : Standard du domaine public
	- Servait uniquement au système NFS (serveur de fichier) de Sun (Linux) à la base
- Dans le programme client, on retrouve une fonction locale qui a le même nom que la distante
-  Serveur ==> Attends les connexions clientes et appeler fonction avec bon paramètres + renvois résultat
- Les fonctions qui prennent en charge les connexions réseau sont des **"stub"** (talons)
- RPCss (Remote Procedure Call Subsystem) - processus générique de Windows NT/2000.XP (DHCP, Messenger...)
- On peut mettre un système de queue

** Avantages / inconvénients**
- Pratique pour les calculs
MAIS 
- Pas de pointeurs 
- Pas de très gros volumes de données (bande passante)

**Créer son propre RPC :**

````C
//calcul.x

struct data {
  unsigned int arg1;
  unsigned int arg2;
};
typedef struct data data;
struct reponse {
  unsigned int somme;
  int errno;
};
typedef struct reponse reponse;
program CALCUL{ // Nom du programme
  version VERSION_UN{ //Nom version
    void CALCUL_NULL(void) = 0; //Procédure 1 - Nom + numéro (0) (Obligatoire)
    reponse CALCUL_ADDITION(data) = 1; //Procédure 2 - 1 seul arg au max donc --> struct
  } = 1; //Numéro version
} = 0x20000001; //Identifiant, ne rentrant pas en conflit avec d'autres programmes
````

- **rpcgen -a calcul.x** //a va permettre de produire un squelette pour programme client et fonction distante.
	- calcul.h // entête
	- calcul_clnt.c //talon client
	- calcul_svc.c //talon serveur
	- calcul_xdr.c //routines XDR - Définit les types
- On écrit la fonction distante sur le serveur //calcul_server.c
	- Compiler le tout
	- Lier ensemble : **gcc** -o server calcul_svc.o calcul_server.o calcul_xdr.o
	- Vérifier le fonctionnement :
`````C
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
`````
- **On écrit la fonction cliente sur le serveur** //calcul_client.c
	- Compiler + lier : 
`gcc -c calcul_client.c`
`gcc -o client calcul_client.o calcul_clnt.o calcul_xdr.o`

{...}Tuto complet ici  http://okki666.free.fr/docmaster/articles/linux116.htm



## 3 - NFS (Network File System):

- Point de montage réseau sur serveur NFS (Pouvoir modifier comme en local) != FTP
- A la base NFS était sous UNIX 
- Sun Microsystems était un des pionniers dans le dév. de UNIX
- Suit le modèle classique TCP IP
- Utilise UDP ⇒ Performances (à travers le réseau) (puis TCP depuis v3)
	- Rendu comme très simple (Pas de traces, requêtes indépendantes des unes des autres...) ⇒ File corruption do not occur and plus facile à gérer en cas de crash sur des cas pour retrouver des données.
	- Le serveur répond juste, le client fait tout le boulot
- Port de base : 2049
- Sorti en V2 en 1989, le standard à évolué jusqu'en V4 jusqu'en 2003 (275 pages)
	- V2/V3 prennent en compte la sécurité avec de l'authentification
	- Selon les versions, de nouvelles procédures RPC (mkdir...)
	- V4 des procédures RPC qui check les droits / chiffrements / Montée en charge / délégation / maintenance / réplication / reprise sous incident / compatibilité
- Inclus quatre composantes pour fonctionner :
	- External Data Representation (XDR) standards : Comment les données sont représentés dans l'échange
		- Traduit dans la langue les données avant de les mettre sur le point de montage réseau
		- Définit "int, bool, float, double, enum, void" {...} / .xcml {...} ==> couche présentation
	- Remote Procedure Call (RPC) : Pour appeler les procédures sur les machines à distance
		- Pour ne pas avoir les instructions sur chaque PC
	- Des opérations et des procédures NFS qui utilisent le RPC
	- Protocole de montage Un modèle basé sur UNIX (mais pas spécifique à lui même): 
		- arrangement hiérarchique des répertoires (/etc/hosts == C:\Windows\Hosts)
		- portion disponible pour le client
 ![](http://www.tcpipguide.com/free/diagrams/nfscomponents.png)
 
Les NAS ⇒ Equipements physiques qui peuvent utiliser NFS

**Opérations** 
````
Procedure 0:  NULL - Do nothing 
Procedure 1:  GETATTR - Get file attributes 
Procedure 2:  SETATTR - Set file attributes 
Procedure 3:  LOOKUP - Lookup filename 
Procedure 4:  ACCESS - Check Access Permission 
Procedure 5:  READLINK - Read from symbolic link 
Procedure 6:  READ - Read From file 
Procedure 7:  WRITE - Write to file 
Procedure 8:  CREATE - Create a file 
Procedure 9:  MKDIR - Create a directory 
Procedure 10: SYMLINK - Create a symbolic link 
Procedure 11: MKNOD - Create a special device 
Procedure 12: REMOVE - Remove a File 
Procedure 13: RMDIR - Remove a Directory 
Procedure 14: RENAME - Rename a File or Directory 
Procedure 15: LINK - Create Link to an object 
Procedure 16: READDIR - Read From Directory 
Procedure 17: READDIRPLUS - Extended read from directory 
Procedure 18: FSSTAT - Get dynamic file system information 
Procedure 19: FSINFO - Get static file system Information 
Procedure 20: PATHCONF - Retrieve POSIX information 
Procedure 21: COMMIT - Commit cached data on a server to stable storage
````


## 4 - DFS (Distributed File System) :
- Système distribué de Microsoft Windows NT & Server
- Mis sur l'AD 
- Accessible via le protocole SMB (Server Message Block), dans un réseau distribué
- Virtualise les serveurs sous forme de disque (drag and drop pour envoyer vers un serveur)
- Simplifie la migration de données : Pas besoin de connaître nom serveur
- Sécurité du NTFS
- Permet de réduire les pb liés à la gestion complexe des config serveur en évolution permanente, quand anciens serveurs supprimés
- On peut construire une hiérarchie comme on le souhaite avec les données de chaque serveur
-    Deux composants: 
	- Transparence de la localisation (pas le nom du serveur)
	- Redondance : Réplication du composant sur plusieurs systèmes

**Important**
- Racine : Point entrée
- Dossiers : Dossier qui permet accéder
- Cible : Serveur sur lequel sont stockés les données
## 5 - Lexicographie - IDL :

- Sun ONC/RPC : Open Network Computing / Remote Procedure Call ((Procédure distante))
- TI-RPC : == Sun RP
- **OSF DCE** - Open Software Foundation (OSF) a développé DCE (Distribued Computing Environment), fournissant un framework et un toolkit pour les applications client/serveur (dont un RPC, un répertoire, temps de service, un service d'authentification et le **DFS**)
- OMG CORBA - Common Object Request Broker Architecture (CORBA) est un standarad définit par l'Object Management Group (OMG) pour facilité la communication des systèmes sur plusieurs plateformes, en utilisant un modèle orienté objet.
- Sun Java RMI : <==> RPC mais version java (pas la même chose)
- Sun J2EE EJB : Applications distribués (J2EE ==> web)
- WS-SOAP : Web service - extension de SOAP (enveloppe), définissant l'intégrité et la confidentialité durant les communications (kerberos...)

## 6 - Risques & problèmes :

- Difficulté pour détecter les défaillances : propriété d’asynchronisme
- Dynamisme : Composition du système change en permance 
	- Pas d'état global
	- difficultés pour administrer le système
- Grande taille : Composants, utilisateurs, dispersion géographique...
	- Capacité de croissance difficile à réaliser

Système répartis qui ont réussis à s'adapter :
- World Wide Web
- DNS 

## 7 - Principes & autres méthodes d'appels de proc distantes:

**Stratégies d'appel procédure distante  **
- **migration** : Code et données envoyés sur le serveur pour exécuter en local directement
	- Statégie de pré-chargement en mémoire
	- Très efficace pour de nombreux appels
	- Problèmes de partage d'objet
	- Mauvaise performances selon le volume de code et de données

- **mémoire partagée répartie** : Procédure dans mémoire partagée entre client/serveur (enfaite dans espace réel du serveur)
	- Efficace si tout le code et les données ne sont pas visités
	- Pas de problème de pointeur
	- Mais volumes de code et de données à échanger pages par pages
- **Par message Asynchrone** : RPC bad 
- **Par appel léger :** RPC 
