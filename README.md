
### Mots Clés
-   IPC*
-   Fichier de Log*
-   RPC*
-   NFS / DES*
-   Socket*
-   Couche IP
-   Système de fichier
-   Get / Post
-   Fichier sur un réseau
-   Gestion des droits d’accès
-   Protocole
-   Communication inter-processus*
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

