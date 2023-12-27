# Mise en prod

## Connexion :

Créer une MV sur Kourou : 
`https://kourou.oclock.io/ressources/vm-cloud/`

Se connecter à notre VPS (serveur virtuel) :
`ssh student@flore-oclock-server.eddi.cloud` (remplacer avec votre url !!)

(répondre `yes` à la question qui demande si on peut faire confiance à cette machine)

pour sortir : `exit`

## Installation de l'env de prod

### Node

`node -v` = vérifier la version de node (il n'est pas installé pour l'instant)

On installe un gestionnaire de version de node : NVM
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash`

Et on active NVM : `. ~/.nvm/nvm.sh` (il ne se passe rien, c'est normal !)

On utilise NVM pour installer la dernière version de Node.js :
`nvm install node`

Puis on teste pour vérifier la version : `node -v`

### Autre outils nécessaires 

`apt` = package manager : permet d'installer les outils nécessaires sur notre machine
(pas comme `npm install` qui permet d'installer les dépendences d'un projet donné)

`sudo apt update` = s'assurer que `apt` est à jour et utilise les bon packages

`sudo apt install postgresql postgresql-contrib` = on installe les outils nécessaires

pour vérifier : `psql --version`

### Récupérer le code de notre projet

`git clone git@github.com:O-clock-Cheesecake/oKanban-backend-Ghislain.git`

=> `are you sure you wat to conrinue...` --> yes
=> `fatal: Could not read from remote repository. Please make sure you have the correct access rights`
=> on n'a pas les droits...

BESOIN : que notre serveur puisse communiquer avec Github (pour récup notre code)
PROBLEME : notre serveur n'a pas les droits d'accès à notr repo Github
SOLUTION: pour que Github accepte de répondre à serveur, il faut lui fournir la clé publique de notre serveur = clé SSH

### Gérerer une clé SSH pour Github

`ssh-keygen -t ed25519 -C "your_email@example.com"` = créer une clé SSH sur notre serveur (en utilisant l'email comme label) (puis 3x touhe `entrée`)

On va maintenant utiliser la clé publique comme "Deploy key" pour notre porojet sur Github :
- aller sur le repo à deployer
- Settings > Deploy Keys (dans le menu à gauche) > Boutton "Add deploy key" (en haut à droite)
- on remplit le title : "okanban VPS
- key : ici il faut coller la clé publique SSH (elle commence par ssh... et termine par votre email => il faut tout copier)

Pour copier la clé publique SSH : `cat /home/student/.ssh/id_ed25519.pub`

### Installer notre application

Récupérer le code source depuis Github : `git clone git@github...`

POur vérifier : `ls` => le dossier est ien là !!

On se déplace dans le dossiper du projet `cd nomDuRepo`

On installe les dpendances : `npm i` (installe toutes les dépendances du projet qui se trouvent dans le package.json)

On créer un fichier `.env` avec les variables d'env :

Rappel des commandes pour manipuler des fichiers : 
- `touch nomDuFichier` = créer un nouveau fichier
- `mkdir nomDuDossier` = créer un nouveau dossier (on n'en a pas besoin nous là)
- `ls -a` = voir tous les fichiers (y compris les cachés) : pour vérifier qu'on a bien créé notre fichier `.env`
- (si on avait eu un fichier `.env.example` on aurait pu le copier avec la commande `cp`)
- `cat nomDuFichier` = lire le contenu d'un fichier (pour l'instant notre .env est vide)
- `nano nomDuFichier` = modifier le contenu d'un fichier (pour enregistrer et sortir : `Ctrl X`, puis `y`, puis `Enter`)

On mets les informations suivantes dans le fichier `.env` :

PORT=3000
PG_URL=postgresql://okanban_admin:okanban@localhost:5432/okanban


### Créer la BDD

[Fiche Recap](https://kourou.oclock.io/ressources/fiche-recap/postgresql/)

=> on choisis les mêmes informations que celle qu'on a érit dan le `.env` (nom d'utilisateur, password, nom de database)

- `psql --version` (l faut avoir installé psql)
- `sudo -i -u postgres`
- `psql`
- `CREATE ROLE okanban_admin WITH LOGIN PASSWORD 'okanban';`
- `CREATE DATABASE okanban OWNER okanban_admin;`
- pour quitter : `exit` (2x)


### Configurer postgres

2 types de connexions avec postgres : 
`peer` = avec la session utilisateur (par default, mais c'est pas ce qu'on veut)
`md5` = avec le mot de passe (c'est ce type de connexio qu'on veut utiliser)

Editons le fichier de config de postgres :
`sudo nano /etc/postgresql/12/main/pg_hba.conf`

Scroller tout en bas du fichier et modifuier la ligne suivante :
`local   all   all   peer`
devient : 
`local   all   all   md5`

pour save avec nano : `Ctrl O` (puis `Enter` pour garder le même nom de fichier). 
puis pour sortir : `Ctrl + X`

On a changé la config de postgres donc il faut le relancer :
`sudo service postgresql restart` (rien ne s'affiche, mais normalement c'est bon, ça a fonctioné)

Pour vérifier : on se connecter à notre db : 
`psql -U okanban_admin -d okanban` (+ password : okanban). Si on a `okanban=> ` c'est que c'est ok !!

### Remplir la database :

- `cd data`
- `psql -U okanban_admin -d okanban -f create_tables.sql`
- `psql -U okanban_admin -d okanban -f seed_database.sql`

Pour vérifier : `psql -U okanban_admin -d okanban` (+ password : okanban), 
puis : `\d` => on retrouve nos tables, c'est tout bon ! 

### Lancer notre projet !

(on ressort du dossier data : `cd ..`)

On lance le fichier d'index de notre projet : `node index.js`

Pour se connecter, je vais sur mon navigateur : `http://pseudo-github-server.eddi.cloud:3000/` 


PROBLEME : la base_url n'est pas la bonne.

- pour la modifier : `nano assets/index-0084f2b2.js`
- pour chercher dans la page : `Ctrl W` "base_url"
- remplacer par `http://flore-oclock-server.eddi.cloud:3000`
- (pour enregistrer et sortir : `Ctrl X`, puis `y`, puis `Enter`)

Note : idéalement il ne faudrait pas le mettre en dur comme çça, mais avoir uhne variable d'env qui changera en fonction de si on est en dev ou en prod (dans le fichier .env) (mais on va pas faire ça maintenant)

- et enfin on relance le fichier index.js : `node index.js` => ça maaaarche !!!