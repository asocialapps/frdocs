---
layout: page
title: Design des "serveurs", leur API
---

**EN REVISION  cette documentation n'est pas totalement alignée avec le code actuel**

Ce design a été conçu afin de pouvoir opter entre plusieurs schémas de déploiement selon l'option de configuration choisie:
- un serveur HTTP unique assurant les services OP+PUBSUB,
- des serveurs ou des instances de Cloud Function assurant le service OP,
- un serveur ou une instance de Cloud Function assurant le service PUBSUB.

Les sources sont hébergés dans github sous le nom de projet `asocial-srv`

### Installations prérequises
- `git`
- `node.js` (donc `npm`) par `nvm`, 
- `yarn`
- `SQLite` : `sqlite(.exe)` est utilisé pour effectuer des backup / restore
- `DBBrowser for SQLite`  pour charger le schéma, exécuter des scripts, parcourir les tables ...
- `VSCode`

- `firebase CLI` : pour les tests en Firestore / Storage de GCP.
- `minio` : pour les tests S3.

## Structure du folder de développement
`src/*`
- contient les modules de l'application détaillés ci-après.

`keys.json`
- données confidentielles de configuration hors git.

`index.js`
- amorce du service OP quand il est géré en _Cloud function_.

`index2.js`
- amorce du service PUBSUB quand il est géré en _Cloud function_.

**Folders / fichiers liés à VSCode**
- `.vscode .yarn .yarnrc.yml .editorconfig .eslintignore .eslintrc.cjs`

**Folders / fichiers de données de test**
- `keys`: **hors git**, contient le certificat HTTPS du serveur en test `fullchain.pem privkey.pem` renouvelés assez fréquemment.
- `fsstoragea/* fsstorageb/*`: storage de test en File-System.
- `sqlite`: base de données de test SqLite:
  - `schema.sql` : script de création d'initialisation de la base.
  - `delete.sql` : script de reset des tables.
  - `testa.db3 testb.db3` et 2 fichiers `wal` associés, bases de 2 environnements de test en sqlite.
  - `test1a.bk test2a.bk ...` : backups de la base de l'envrionnement 'a' dans des états privilégiés '1' et '2'
  - `bkp.sh bkp.ps1` : shell scripts de backups de la base courante en `testXY.bk`
  - `rst.sh rst.ps1` : shell scripts de restauration d'un textXY.bk sur la base courante.

`emulators/` : **Fichiers liés au test / déploiement de Firestore**
- `bk1/* bk2/*`: sauvegardes des états internes des exécutions d'émulation Firebase / Storage (GCP).
- `.firebaserc` : afin que le client Firebase n'exige pas le code du projet.
- `firebase.json` : configuration de test local
- `firebase-debug.log ui-debug.log`
- `firestore.indexes.json` : index Firestore (source).
- `firestore.indexes.EXP.json` : index Firestore _exporté_ de la base réelle.
- `firestore.rules` : droits d'accès à Firestore.
- `storage.rules` : droits d'accès au Storage.

`gae/` : **Fichiers pour le déploiement en GAE**
- `.gcloudignore` : fichiers que gcloud ne doit pas importer lors du déploiement.
- `app.yaml` : descriptif du déploiement de l'application en GAE.
- `config.mjs` : configuration pour GAE (issu de src/config.mjs).
- `cron.yaml` : condition de lancement du GC quotidien.
- `depl.ps1 depl.sh` : scripts de recopie de fichiers dans le folder de de déploiement GAE.

**Fichier de configuration de minio pour tests S3 locaux**
- `minio.json`

**Autres folders / fichiers**
- `node_modules/*`
- `.gitignore` : fichiers à ne pas gérer dans git;
- `package.json`
- `yarn.lock`
- `webpack.config.mjs` : pour génération d'un distribuable.
- `README.md`
- `etc/*` : quelques fichiers de diverses utilités (dont `minio.json`).
- `tmp/` : folder temporaire de convenance..

## Modules du serveur
### Configuration: `src/config.mjs src/secret.mjs keys.json src/gensecret.mjs`
La configuration _de base_ (options par défaut, mais incomplète) est inscrite dans `src/config.mjs`.

Elle est surchargée à l'exécution par la configuration spécifique donnée par l'administrateur technique exprimée dans `keys.json` qui contient:
- les clés d'authentification d'accès aux services externes (Google Firestore / Storage), S3, de connexion aux bases ...
- le hash de la clé d'authentification de l'administrateur technique,
- la clé de cryptage des données dans la base,
- les clés publique / privée VAPID de notification web-push.

En conséquence en raison de la confidentialité requise pour ces données sensibles, `keys.json` n'est pas disponible dans le dépôt git du serveur.
- en développement / déploiement, ce fichier est _crypté_ et devient le fichier `src/secret.mjs`
- au lancement de l'application serveur, `config.mjs` importe ce fichier, en décrypte le contenu et installe toutes ses entrées dans l'objet `config` du module `config.mjs`
- l'objet `config` est importé ensuite dans les autres modules en ayant besoin.

Le cryptage de `keys.json` en `src/secret.mjs` est assuré par la commande :

    node src/gensecret.mjs

### Utilitaires
`src/api.mjs`
- module comprenant des constantes et des classes / fonctions utilitaires devant être strictement identiques entre le serveur et l'application Web.

`src/base64.mjs`
- module lui aussi identique en application Web et serveur, principalement pour uniformité d'API utilisé dans d'autres modules(l'application Web n'a pas accès à node).

`src/logger.mjs`
- gestion du log par Winston.

`src/util.mjs`
- fonctions utilitaires diverses (dont encryption).

### Module d'accès à la base de données
Les modules présentent exactement le même API externe: les autres modules ignorent quelle implémentation est utilisée, le choix étant fixé à la configuration.
- `src/dbSqlite.mjs`
- `src/dbFirestore.mjs`

### Modules d'accès au _Storage_
Les modules présentent exactement le même API externe: les autres modules ignorent quelle implémentation est utilisée, le choix étant fixé à la configuration.
- `src/storageFS.mjs` : storage sur File-System local
- `src/storageGC.mjs` : storage Google Cloud Storage
- `src/storageS3.mjs` : storage AWS/S3

### Outil en CLI
`src/tools.mjs`
- propose les commandes: `export-db export-fs purge-db purge-fs`
- la commande `vapid` génère un couple de clé publique / privée VAPID pour le push-web. La clé publique est à communiquer à la configuration de l'application Web.

### Module du service PUBSUB
`src/pubsub.mjs`
- enregistre les connexions des sessions, leurs _heartbeat_, leurs déconnexions.
- reçoit les avis de mise à jour des documents après chaque opération,
- publie aux sessions les avis de mise à jour des documents de leur périmètre.

### Amorce des _serveurs_ et _cloud functions_
`src/server.js`
- amorce d'un _serveur_ assurant les services OP+PUBSUB.

`index.mjs`
- amorce _Cloud function_ d'un service OP seul.

Ces deux amorces importent `src/cfgexpress.mjs` qui configure express pour les URLs d'entrée.

`src/pubsub.js`
- amorce d'un _serveur_ n'assurant que le service PUBSUB.

`index2.mjs`
- amorce _Cloud function_ d'un service PUBSUB seul.

Ces deux amorces importent `src/cfgexpress2.mjs` qui configure express pour les seules URLs d'entrée du service PUBSUB.

`src/cfgexpress.mjs`
- configure les URLs d'entrée.
- l'URL `/op/...` invoque une fonction operation qui,
  - instancie un objet de classe `Operation` correspondant à l'opération souhaitée,
  - l'initialise,
  - invoque sa méthode `run(args, dbp, storage)`,
    - arguments de la requête, providers d'accès à la base et provider du storage 
  - retourne son résultat comme retour du POST de la requête qui l'a déclenché,
  - récupère ses exceptions et gère le retour correspondant de la requête.

`src/cfgexpress2.mjs`
- configure les URLs d'entrée
- l'URL `/pubsub/...` invoque la fonction pubsub de `notif.mjs`.

### Service PUBSUB: module `src/notif.mjs`
Ce module expose trois entrées:
- `genLogin` : invoqué au login d'une session.
- `genNotif` : invoqué à une fin d'opération.
- `genHeartbeat` : invoqué par un appel heartBeat d'une application Web.

Ce module a la caractéristique d'avoir deux modes d'appel:
- **en simple appel de fonction**: dans un serveur OP+PUBSUB, le service OP appelle les méthodes `genLogin` `genNotif`.
- **en réception de requête HTTP**: dans un serveur PUBSUB ou une _Cloud function PUBSUB_ les appels sont reçus par requête HTTP.

Quand `notif.mjs` est invoqué sur l'une des fonctions `genLogin` ou `genNotif`,
- s'il détecte qu'il est mode _serveur OP+PUBSUB_ il invoque directement la méthode correspondante,
- sinon il génère une requête POST à destination du _serveur / Cloud function_ PUBSUB.

Ainsi vu du service OP, le fait que PUBSUB soit ou non hébergé dans le même serveur est opaque.

Les commentaires et le code de `notif.mjs` sont suffisants pour le comprendre (400 lignes de code).

### Les 5 modules du service OP
`src/modele.mjs` : documentation détaillée ci-après:
- classe `Cache` : gestion d'une mémoire cache des documents majeurs.
- classe `TrLog` : gère l'objet de communication d'une opération au service PUBSUB.
- classe `GD` : pour chaque opération un objet de classe GD est un gestionnaire d'une mémoire cache des documents lus et modifiés par l'opération AVANT son commit.
- classe `Operation` : gestion des phases d'exécution de l'opération: authentification, phase 1, phase 2 transactionnelle, phase 3, fin d'opération.

`src/gendoc.mjs`
- une classe `Document` (générique) à des méthodes applicables à tous les documents.
- chaque sous-classe correspond à un `Document` et comporte les méthodes de traitement spécifiques associés.
- les commentaires dans le code sont suffisants pour la compréhension.

`src/taches.mjs`
- les _tâches_ sont opérations qui ont été déclenchées par une opération mais s'exécutent en différé après la fin de l'opération l'ayant initiée. C'est la fonction de la classe `Tache` (commentée dans le code).
- le module contient aussi la méthode correspondante aux seules opérations différées. Elles sont commentées dans le code.

`src/operations3.mjs src/operations4.mjs`
- ces 2 modules ne sont distincts que pour des raisons de taille. 
- `operations3.mjs` est réservé aux opérations spéciales, création / gestion d'espace, de compte, synchronisation, etc.
- `operations4.mjs` correspond à toutes les autres opérations _normales_.
- chaque opération a une classe héritant de `Operation`:
  - son API (ses arguments) est documentée,
  - des commentaires explicitent le traitement.

La logique applicative est répartie entre:
- les classes Operation (`taches.mjs operations3.mjs operations4.mjs`),
- les classes Document (`gendoc.mjs`)

-----------------------------------------------------------

# API : opérations supportées par le serveur

Les opérations sont invoquées sur l'URL du serveur : `https://.../op/MonOp1`
- **GET** : le vecteur des arguments nommés sont dans le queryString :
  `../op/MonOp1&arg1=v1&arg2=v2 ...`
  Ils se retrouvent à disposition dans l'opération dans l'objet `{ arg1: v1, arg2: v2 }`
- **POST** : le body de la requête est la sérialisation de `{ arg1: v1, arg2: v2 ... }`

### `args.token` : le jeton d'authentification du compte
Requis dans la quasi totalité des requêtes ce jeton est formé par la sérialisation de la map `{ sessionId, shax, org, hps1 }`:
- `sessionId` : identifiant aléatoire (string) de session générée par l'application pour s'identifier elle-même (et transmise sur WebSocket en implémentation SQL).
- `org` : code de l'organisation.
- `shax` : SHA256 du PBKFD de la phrase secrète.
- `hps1` : Hash (sur un entier _safe_ en Javascript) du SHA256 de l'extrait de la phrase secrète.

L'extrait d'une phrase secrète consiste à prendre les 12 premiers caractères de la phrase complète.

### Headers requis
- `origin` : site Web d'où l'application Web a été chargée. Au cas ou `origin` ne peut pas être obtenu, c'est `referer` qui est considéré. Les origines autorisées sont listées dans `config.mjs`.
- `x-api-version` : numéro de version de l'API pour éviter un accès par des sessions ayant été chargées par une application _retardée_ par rapport au serveur.

### Retour des requêtes
- status HTTP 200 : OK
- status HTTP 400 401 402 403 : exceptions trappées par le serveur ,:
  - 400 : F_SRV fonctionnelles
  - 401 : A_SRV assertions
  - 402 : E_SRV exception inattendue trappée dans le traitement
  - 403 : E_SRV exception inattendue NON trappée dans le traitement
  - le texte de l'erreur est un texte JSON : `{ code, args: [], stack }`
  - à détection au retour d'une requête, une exception `AppExc` est générée depuis ces données.
- autre statuts (500, 0 ...) : une exception `AppExc` est générée (E_SRV, 0, ex.message)

#### Retour OK d'un GET
- requêtes `/op/yo` et `/op/yoyo` : texte. 
  - `yo` est traitée _avant_ contrôle de l'origine et retourne `yo`.
  - `yoyo` est traitée _après_ contrôle de l'origine et retourne `yoyo`.
- autres requêtes : `arrayBuffer` (binaire).

#### Retour OK d'un POST
Le résultat binaire est désérialisé, on en obtient une map dont les éléments sont :
- de manière générique :
  - `adq` : le record de synthèse de comptabilité après l'opération.
  - `trLog` : le record de notification web-push contenant les indications de mises à jour du périmètre du compte.
  - `nhb` :  c'est le couple { sessionId, nhb } :
    - `sessionId` passé dans le token en argument.
    - `nhb` : numéro de _heartbeat_ courant dans le serveur PUBSUB pour vérification de séquence.
- les autres termes sont spécifiques du _retour_ de chaque opération quand il y en a.

### Synthèse des URLs traitées
OPTIONS `/*`
- toutes les URLs sont concernées. Ne retourne rien mais est systématiquement invoquée par les browsers pour tester les accès cross-origin.

GET `/favicon.ico`
- retourne la favicon de l'application spécifiée en configuration.

GET `/robots.txt`
- retourne `'User-agent: *\nDisallow: /\n'`

GET `/ping`
- retourne en string la date et l'heure UTC.

GET `/storage/...`
- utilisé pour télécharger un fichier quand le provider de storage est `fs` (ou `gc` en mode simulé). Retourne le fichier crypté en binaire `application/octet-stream`.

PUT `/storage/...`
- utilisé pour uploader un fichier quand le provider de storage est `fs` (ou `gc` en mode simulé). Le fichier est crypté en binaire `application/octet-stream` dans le `body` de la requête.

POST `/op/...`
- les `opérations` de l'application détaillées ci-après.

POST `/pubsub/...`
- les opérations de PUBSUB quand elles parviennent en HTTP.

## Opérations `/op/...`

### Dans `src/operations3.mjs`

#### EchoTexte: Écho du texte envoyé

    to: { t: 'int', min: 0, max: 10 },
    texte: { t: 'string' }

Retour:
- `echo` : texte d'entrée retourné.

#### ErreurFonc: Erreur fonctionnelle simulée du texte envoyé

    to: { t: 'int', min: 0, max: 10 },
    texte: { t: 'string' }

Exception: A_SRV, 10

#### PingDB: Test d'accès à la base - GET
Insère un item de ping dans la table singletons/1

Retour:
- un `text/plain` avec les date-heures de ping (le précédent et celui posé).

#### GetEspaces : pour admin seulement, retourne tous les rows espaces
Retour:
- `espaces` : array de rows `espaces`.

#### GetPub: retourne la clé RSA publique d'un avatar

    id: { t: 'ida' } // id de l'avatar

Retour:
- `pub`: clé RSA publique de l'avatar.

#### GetPubOrg: retourne la clé RSA publique d'un avatar NON authentifié

    org: { t: 'org'}, // code de l'organisation
    id: { t: 'ida' }  // id de l'avatar

Retour:
- `pub`: clé RSA publique.

#### GetSponsoring : obtention d'un sponsoring par le hash de sa phrase

    org: { t: 'org'},
    hps1: { t: 'ids' }, // hash9 du PBKFD de la phrase de contact réduite
    hTC: { t: 'ids' } // hash de la phrase de sponsoring complète

Retour:
- `rowSponsoring` s'il existe

#### ExistePhrase1: Recherche hash de phrase de connexion

  org: { t: 'org'},
  hps1: { t: 'ids' } // hash9 du PBKFD de la phrase de contact réduite

Retour:
- `existe` : `true` si le hash de la phrase existe.

#### ExistePhrase: Recherche hash de phrase

    t: { t: 'int', min: 2, max: 3 },
    // 2 : phrase de sponsoring (ids)
    // 3 : phrase de contact (hpc d'avatar)
    hps1: { t: 'ids' } // hash9 du PBKFD de la phrase de contact réduite

Retour:
- `existe` : `true` si le hash de la phrase existe.

#### AcceptationSponsoring - synchronisation sur ouverture d'une session à l'acceptation d'un sponsoring

    org: { t: 'org' }, // organisation
    idsp: { t: 'ida' }, // identifiant du sponsor
    idssp: { t: 'ids' }, // identifiant du sponsoring
    id: { t: 'ida' }, // id du compte sponsorisé à créer
    hXR: { t: 'ids' }, // hash du PBKD de sa phrase secrète réduite
    hXC: { t: 'ids' }, // hash du PBKD de sa phrase secrète complète
    hYC:  { t: 'ids' }, // hash du PNKFD de la phrase de sponsoring
    cleKXC: { t: 'u8' }, // clé K du nouveau compte cryptée par le PBKFD de sa phrase secrète complète
    cleAK: { t: 'u8' }, // clé A de son avatar principal cryptée par la clé K du compte
    ardYC: { t: 'u8' }, // ardoise du sponsoring
    dconf: { t: 'bool' }, // dconf du sponsorisé
    pub: { t: 'u8' }, // clé RSA publique de l'avatar
    privK: { t: 'u8' }, // clé privée RSA de l(avatar cryptée par la clé K du compte
    cvA: { t: 'cv' }, // CV de l'avatar cryptée par sa clé A
    clePK: { t: 'u8', n: true }, // clé P de sa partition cryptée par la clé K du nouveau compte
    cleAP: { t: 'u8', n: true }, // clé A de son avatar principâl cryptée par la clé P de sa partition
    clePA: { t: 'u8', n: true }, // cle P de la partition cryptée par la clé A du nouveau compte
    ch: { t: 'chsp', n: true }, // { ccK, ccP, cleE1C, cleE2C, t1c, t2c }
      // ccK: clé C du chat cryptée par la clé K du compte
      // ccP: clé C du chat cryptée par la clé publique de l'avatar sponsor
      // cleE1C: clé A de l'avatar E (sponsor) cryptée par la clé du chat.
      // cleE2C: clé A de l'avatar E (sponsorisé) cryptée par la clé du chat.
      // t1c: mot du sponsor crypté par la clé C
      // t2c: mot du sponsorisé crypté par la clé C
    htK: { t: 'u8' }, // hashtag relatif au sponsor
    txK: { t: 'u8' } // texte relatif au sponsor

Retour:

#### RefusSponsoring: Rejet d'une proposition de sponsoring

    org: { t: 'org'},
    id: { t: 'ida' }, // identifiant du sponsor
    ids: { t: 'ids' }, // identifiant du sponsoring
    ardYC: { t:'u8' }, // réponse du filleul
    hYC: { t: 'ids' } // hash9 du PBKFD de la phrase de contact réduite

#### GetSynthese : retourne la synthèse de l'espace ns ou courant

    org: { t: 'org', n: true } // id de l'espace (pour admin seulement, sinon c'est celui de l'espace courant)

Retour:
- `rowSynthese`

#### GetPartition : retourne une partition

    id: { t: 'idp', n: true } // id de la partition

Retour:
- `rowPartition`

#### GetEspace : retourne certaines propriétés de l'espace

Et enregistre la notification éventuelle dans la compta du demandeur.

Retour:
- rowEspace s'il existe


#### Sync : opération générique de synchronisation d'une session cliente

    subJSON: { t: 'string', n: true }, // subscription de la session
    nhb: { t: 'int', n: true}, // numéro de HB pour un login / relogin
    nbIter: { t: 'int' },
    dataSync: { t: 'u8', n: true }, // sérialisation de l'état de synchro de la session
    // null : C'EST UNE PREMIERE CONNEXION - Création du DataSync
    // recherche des versions "base" de TOUS les sous-arbres du périmètre, inscription en DataSync
    lids: { t: 'lids', n: true }, // liste des ids des sous-arbres à recharger (dataSync n'est pas null)
    full: { t: 'bool', n: true } // si true, revérifie tout le périmètre

LE PÉRIMÈTRE est mis à jour: `DataSync` aligné OU créé avec les avatars / groupes tirés du compte.

Retour:
- `datasync`
- `rowGroupes rowMembres rowChatgrs`
- `rowNotes`
- `rowAvatars rowChats rowSponsorings rowTickets`
- `rowEspace tarifs`
- `rowCompti rowInvit rowCompte`

#### Adq : cette opération ne fait rien mais force le retour du record `adq`

#### SetEspaceQuotas: Déclaration des quotas globaux de l'espace par l'administrateur technique

    org: { t: 'org' }, // id de l'espace modifié
    quotas: { t: 'q' } // quotas globaux

#### SetNotifE : déclaration d'une notification à un espace par l'administrateur

    org: { t: 'org' }, // id de l'espace notifié
    ntf: { t: 'ntf' } // sérialisation de l'objet `notif`, cryptée par la clé du comptable de l'espace. Cette clé étant publique, le cryptage est symbolique et vise seulement à éviter une lecture simple en base

#### GetNotifC : obtention de la notification d'un compte
Réservée au comptable et aux délégués de la partition du compte

    id: { t: 'ida' } // id du compte dont on cherche la notification

Retour:
- `notif`

#### CreationEspace : création d'un nouvel espace

    org: { t: 'org' }, // code de l'organisation
    TC: { t: 'u8' }, // PBKFD de la phrase de sponsoring du Comptable par l'AT
    hTC: { t: 'ids' } // hash de TC

Traitement ssi: 
- soit espace n'existe pas, 
- soit espace existe et a un `hTC` : re-création avec une nouvelle phrase de sponsoring.

Création des rows `espaces`, `synthese`
- génération de la `cleE` de l'espace: -> `cleET` (par TC) et `cleES` (par clé système).
- stocke dans l'espace: `hTC cleES cleET`. Il est _à demi_ créé, son Comptable n'a pas encore crée son compte.

#### MajSponsEspace : Changement de la phrase de contact du Comptable

    org: { t: 'org' }, // code de l'organisation
    TC: { t: 'u8' }, // PBKFD de la phrase de sponsoring du Comptable par l'AT
    hTC: { t: 'ids' } // hash de TC

#### CreationComptable : création du comptable d'un nouvel espace

    org: { t: 'org' }, // code de l'organisation
    idp: { t: 'idp' }, // ID de la partition primitive
    hTC: { t: 'ids' }, // hash du PBKFD de la phrase de sponsoring du Comptable
    hXR: { t: 'ids' }, // hash du PBKFD de la phrase secrète réduite
    hXC: { t: 'ids' }, // hash du PBKFD de la phrase secrète complète
    pub: { t: 'u8' }, // clé RSA publique du Comptable
    privK: { t: 'u8' }, //  clé RSA privée du Comptable cryptée par la clé K
    clePK: { t: 'u8' }, // clé P de la partition 1 cryptée par la clé K du Comptable
    cleEK: { t: 'u8' }, // clé E cryptée par la clé K
    cleAP: { t: 'u8' }, // clé A du Comptable cryptée par la clé de la partition
    cleAK: { t: 'u8' }, // clé A du Comptable cryptée par la clé K du Comptable
    cleKXC: { t: 'u8' }, //  clé K du Comptable cryptée par XC du Comptable (PBKFD de la phrase secrète complète).
    clePA: { t: 'u8' }, //  cle P de la partition cryptée par la clé A du Comptable
    ck: { t: 'u8' } //  {cleP, code} cryptés par la clé K du Comptable. 
      // cleP : clé P de la partition.
      // code : code / commentaire court de convenance attribué par le Comptable

Création des rows:
- `partitions` : primitive, avec le Comptable comme premier participant et délégué
- `comptes comptis comptas avatars` du Comptable

### Dans `src/operations4.mjs`

#### SetEspaceOptionA : changement de l'option A, nbmi, par le Comptable

    optionA: { t: 'int', min: 0, max: 1, n: true}, // true si accepte le s compte A
    nbmi: { t: 'int', min: 3, max: 18, n: true } // nombre de mois d'inactivité avant suppression d'un compte

#### SetEspaceDlvat : changement de dlvat par l'administrateur

    org: { t: 'org'}, // ns de l'espace concerné
    dlvat: { t: 'date' } // aaaammjj : date limite fixée par l'administrateur technique

#### AjoutSponsoring : Ajout d'un sponsoring

    id: { t: 'ida' }, // id du sponsor
    hYR: { t: 'ids' }, // hash du PBKFD de la phrase réduite de sponsoring,
    psK: { t: 'u8' }, // texte de la phrase de sponsoring cryptée par la clé K du sponsor.
    YCK: { t: 'u8' }, // PBKFD de la phrase de sponsoring cryptée par la clé K du sponsor.
    hYC: { t: 'ids' }, // hash du PBKFD de la phrase de sponsoring
    cleAYC: { t: 'u8' }, // clé A du sponsor crypté par le PBKFD de la phrase complète de sponsoring
    partitionId: { t: 'idp', n: true }, // id de la partition si compte "O" 
    cleAP: { t: 'u8', n: true }, // clé A du COMPTE sponsor crypté par la clé P de la partition.
    clePYC: { t: 'u8', n: true }, // clé P de sa partition (si c'est un compte "O") cryptée par le PBKFD de la phrase complète de sponsoring (donne l'id de la partition).
    nomYC: { t: 'u8' }, // nom du sponsorisé, crypté par le PBKFD de la phrase complète de sponsoring.
    cvA: { t: 'cv' }, // CV { id, v, ph, tx } du sponsor, (ph, tx) cryptés par sa cle A
    ardYC: { t: 'u8' }, // ardoise de bienvenue du sponsor / réponse du sponsorisé cryptée par le PBKFD de la phrase de sponsoring.
    htK: { t: 'u8' }, // hashtag attribué par le sponsor au sponsorisé (crypté cmlé K)
    txK: { t: 'u8' }, // texte attribué par le sponsor au sponsorisé (crypté cmlé K)
    quotas: { t: 'q' }, // quotas {qc, qn, qv} attribués par le sponsor
    don: { t: 'int', min: 1, max: 1000, n: true }, // montant du don pour un compte autonome sponsorisé par un compte autonome
    dconf: { t: 'bool' }, // true, si le sponsor demande la confidentialité (pas de chat à l'avcceptation)
    del: { t: 'bool', n: true }, // true si le compte est délégué de la partition

#### ProlongerSponsoring : prolongation d'un sponsoring existant

    id: { t: 'ida'}, // identifiant de l'avatar du du sponsoring
    ids: { t: 'ids' }, // identifiant du sponsoring
    dlv: { t: 'date', n: true } // nouvelle date limite de validité `aaaammjj`ou 0 pour une  annulation.

#### GetCompta : retourne la compta d'un compte
Le demandeur doit être:
- le comptable,
- OU un délégué de sa partition si c'est un compte O
- OU avec un chat ayant un "mut" avec le demandé si c'est un compte A.

Retour: `rowCompta` s'il existe

    id: { t: 'ida' }, // id du compte dont la compta est demandée
    ids: { t: 'ids', n: true }

#### GetComptaQv : retourne les compteurs qv de compteurs de la compta d'un compte
Retour: `comptaQV`: rowCompta

    id: { t: 'ida' } // id du compte

#### GetAvatarPC : Récupération d\'un avatar par sa phrase de contact
Retour:
- `cleAZC` : clé A cryptée par ZC (PBKFD de la phrase de contact complète)
- `cvA`: carte de visite cryptée par sa clé A
- `collision`: true si la phrase courte pointe sur un  autre avatar

    hZR: { t: 'ids' }, // hash de la phrase de contact réduite
    hZC: { t: 'ids' } // hash de la phrase de contact complète

#### OperationCh : super classe abstraite permettant d'utiliser les méthodes intro1/2()

#### NouveauChat : Création d'un nouveau chat
Retour:
- `rowChat` : row du chat I.

    idI: { t: 'ida' }, // id de l'vatar du chat "interne"
    idE: { t: 'ida' }, // id de l'vatar du chat "externe"
    urgence: { t: 'bool', n: true }, // chats ouvert d'urgence
    mode: { t: 'modech' }, 
    // - 0: par phrase de contact - hZC en est le hash
    // - 1: idE est Comptable
    // - 2: idE est délégué de la partition de idI
    // - idg: idE et idI sont co-membres du groupe idg (idI a accès aux membres)
  
    // 0: par phrase de contact (hZC en est le hash),  
    // 1: idE est délégué de la partition de idI, 
    // idg: idE et idI sont co-membres du groupe idg (idI a accès aux membres)

    hZC : { t: 'ids', n: true }, // hash du PBKFD de la phrase de contact compléte pour le mode 0
    ch: { t: 'nvch' }, // { cck, ccP, cleE1C, cleE2C, t1c }
    // ccK: clé C du chat cryptée par la clé K du compte de idI
    // ccP: clé C du chat cryptée par la clé publique de idE
    // cleE1C: clé A de l'avatar E (idI) cryptée par la clé du chat.
    // cleE2C: clé A de l'avatar E (idE) cryptée par la clé du chat.
    // txt: item crypté par la clé C

#### MutChat : Ajout ou suppression d'une demande de mutation sur un chat 

    id: { t: 'ida' }, // id de l'avatar du chat (principal)
    ids: { t: 'ids' },  // ids du chat
    mut: { t: 'int', min: 0, max: 2 } // type de demande - 1 muter en O, 2 muter en A

#### MajLectChat : mise à jour de la date-heure de lecture du chat ***********

    id: { t: 'ida' }, // id de l'avatar du chat
    ids: { t: 'ids' },  // ids du chat

#### MajChat : Ajout ou suppression d'un item à un chat
Retour:
- `disp`: `true` si E a disparu (pas de maj faite).

    id: { t: 'ida' }, // id de l'avatar du chat
    ids: { t: 'ids' },  // ids du chat
    t: { t: 'u8', n: true }, // texte gzippé crypté par la clé C du chat (null si suppression)
    dh: { t: 'dh', n: true }, // 0 ou date-heure de l'item du chat à supprimer
    urgence: { t: 'bool', n: true }, // chat d'urgence
    don: { t: 'int', min: 0, max: 1000, n: true } // montant du don de I à E

#### PassifChat : Mise en état "passif" d'un chat
Retour
- `disp`: true si E a disparu

    id: { t: 'ida' }, // id de l'avatar du chat
    ids: { t: 'ids' }  // ids du chat

#### ChangementPC : Changement de la phrase de contact d'un avatar
Exceptions:
- F_SRV, 26: Phrase de contact trop proche d'une phrase existante.

    id: { t: 'ida' }, // id de l'avatar
    hZR: { t: 'ids', n: true }, // hash de la phrase de contact réduite (SUPPRESSION si null)
    cleAZC: { t: 'u8', n: true }, //  clé A cryptée par ZC (PBKFD de la phrase de contact complète).
    pcK: { t: 'u8', n: true }, //  phrase de contact complète cryptée par la clé K du compte.
    hZC: { t: 'ids', n: true } // hash du PBKFD de la phrase de contact complète.

#### StatutChatE : Statut du contact d'un chat
Retour: 
- `statut` : `{ cpt, idp, del }`
  - `cpt`: true si avatar principal
  - `idp`: id de la partition si compte "0", 
  - `del`: true si délégué

    ids: { t: 'ids' } // ids = chat

#### RafraichirCvsAv : Rafraîchissement des CVs des membres / chats de l'avatar
Retour: 
- `ncnv` : [nc, nv]
  - `nc`: nombre de CV mises à jour
  - `nv`: nombre de chats / membres scannés

Exception générique:
- 8001: avatar disparu

    id: { t: 'ida' }, // id de l'avatar
    lch: { t: 'array' }, // liste des chats: [{ ids, idE, vcv } ...]
    lmb: { t: 'array' } // liste des membres: [{ id, im, ida, vcv} ...]

#### RafraichirCvsGr : Rafraîchissement des CVs des membres d'un groupe
Retour: 
- `ncnv` : `[nc, nv]`
  - `nc`: nombre de CV mises à jour
  - `nv` : nombre de chats existants

Exception générique:
- 8002: groupe disparu

    idg: { t: 'idg' }, // id du groupe
    lmb: { t: 'array' } // liste des membres: [{ id, im, ida, vcv} ...]

#### SetQuotas: Fixation des quotas d'un compte dans sa partition ou comme compte A

    idp: { t: 'idp' }, // id de la partition
    idc: { t: 'idc' }, // id du compte
    q: { t: 'q' } // quotas: {qc, qn, qv}

#### NouvellePartition : Création d'une nouvelle partition

    idp: { t: 'idp' }, // id de la partition
    itemK: { t: 'u8' }, //  {cleP, code} crypté par la clé K du Comptable.
    quotas: { t: 'q' } // quotas: {qc, qn, qv}

#### SupprPartition : Suppression d'une partition

    idp: { t: 'idp' } // id de la partition

#### SetQuotasPart : Mise à jour des quotas d'une partition

    idp: { t: 'idp' }, // id de la partition
    quotas: { t: 'q' } // quotas: {qc, qn, qv}

#### SetQuotasA : Mise à jour des quotas pour les comptes A

    quotas: { t: 'q' } // quotas: {qc, qn, qv}

#### SetCodePart : Mise à jour du code d'une partition

    idp: { t: 'idp' }, // id de la partition
    etpk: { t: 'u8' } // {codeP, code} crypté par la clé K du Comptable

####  MuterCompteAauto : Auto mutation du compte O en compte A
Mutation d'un compte `c` O de la partition `p` en compte A
- augmente `syntheses.qtA`.
- diminue `partition[p].mcpt[c].q` ce qui se répercute sur `syntheses.tsp[p].qt`.
- bloqué si l'augmentation de `syntheses.qtA` fait dépasser `syntheses.qA`.

    quotas: { t: 'q' } // quotas: { qc, qn, qv }  

####  MuterCompte : classe abstraite

#### MuterCompteA : Mutation du compte O en compte A
Mutation d'un compte `c` O de la partition `p` en compte A
- augmente `syntheses.qtA`.
- diminue `partition[p].mcpt[c].q` ce qui se répercute sur `syntheses.tsp[p].qt`.
- bloqué si l'augmentation de `syntheses.qtA` fait dépasser `syntheses.qA`.

    id: { t: 'ida' }, // id du compte devenant A
    ids: { t: 'ids' }, // ids du chat du compte demandeur (Comptable / Délégué)
    quotas: { t: 'q' }, // quotas: { qc, qn, qv }   
    t: { t: 'u8' } // texte (crypté) de l'item à ajouter au chat

#### MuterCompteO: Mutation d'un compte A en compte O
Mutation d'un compte `c` A en compte O de la partition `p`
- diminue `syntheses.qtA`
- augmente `partition[p].mcpt[c].q` (si c'est possible) ce qui se répercute sur `syntheses.tsp[p].qt`
- blocage si les quotas de la partition ne supportent pas les quotas du compte muté.

    id: { t: 'ida' }, // id du compte devenant O
    quotas: { t: 'q' }, // quotas: { qc, qn, qv }   
    cleAP: { t: 'u8' }, // clé A du compte cryptée par la clé P de la partition
    clePK: { t: 'u8' }, // clé de la nouvelle partition cryptée par la clé publique du compte
    ids: { t: 'ids' }, // ids du chat du compte demandeur (Comptable / Délégué)
    t: { t: 'u8' } // texte (crypté) de l'item à ajouter au chat

#### FixerQuotasA : Attribution par le Comptable de quotas globaux pour les comptes A

    quotas: { t: 'q' } // quotas: { qc, qn, qv } 

#### ChangerPartition : Transfert d'un compte O dans une autre partition

    id: { t: 'ida' }, // id du compte qui change de partition
    idp: { t: 'idp' }, // id de la nouvelle partition
    cleAP: { t: 'u8' }, // clé A du compte cryptée par la clé P de la nouvelle partition
    clePK: { t: 'u8' }, // clé de la nouvelle partition cryptée par la clé publique du compte
    notif: { t: 'ntf', n: true } // notificcation du compte en cours

#### DeleguePartition : Changement de statut délégué d'un compte dans sa partition

    id: { t: 'ida' }, // id du compte qui change de statut
    del: { t: 'bool' } // true / false, statut délégué

#### SetNotifP : notification d'une partition

    idp: { t: 'idp' }, // id de la partition
    notif: { t: 'ntf', n: true } // notification cryptée par la clé de la partition.

#### SetNotifC : notification d'un compte "O"

    idc: { t: 'ida' }, // id du compte
    notif: { t: 'ntf', n: true } // notification du compte cryptée par la clé de partition

#### PlusTicket : Génération d'un ticket de crédit et ajout du ticket au Comptable
Retour: 
- `rowCompta`: du compte après insertion du ticket

    ids: { t: 'ids' }, // ids du ticket généré
    dlv: { t: 'date', n: true },
    ma : { t:'int', min: 0, max: 100000 }, // montant du ticket
    refa: { t: 'string', n: true } // référence éventuelle

#### MoinsTicket: retrait d'un ticket à un compte A et retrait d'un ticket au Comptable
Retour: 
- `rowCompta` : du compte

    ids: { t: 'ids' } // ids du ticket à enlever

#### ReceptionTicket : Réception d'un ticket par le Comptable

    ids: { t: 'ids' }, // ids du ticket reçu
    mc : { t:'int', min: 0, max: 100000 }, // montant du ticket reçu
    refc: { t: 'string', n: true } // référence éventuelle du Comptable

#### MajCv : Mise à jour de la carte de visite d'un avatar ou d'un groupe*/

    cv: { t: 'cv' } // carte de visite (photo / texte cryptés)

#### GetCv : Obtention de la carte de visite d'un avatar OU d'un groupe
- `id` : id du people ou du groupe
- `r` : A quel titre le PEOPLE id est contact du compte ?
  `{ del: true }` : parce que id est délégué de la partition du compte
  `{ id, ids }` : parce qu'il a un chat id / ids avec l'avatar id du compte
  `{ idg, imp, ida, ima }` : parce qu'il est membre d'indice imp du groupe idg
    dont le compte a un avatar ida / ima ayant accès aux membres
- `r` : A quel titre le GROUPE id est visible du compte ?
  `{ ida, ima }`:  parce que l'avatar ida indice ima dans le groupe id a accès aux membres

Retour:
- `cv`: si trouvée

#### NouvelAvatar : Création d'un nouvel avatar du compte

    id: { t: 'ida' }, // id de l'avatar à créér
    cleAK: { t: 'u8' }, // sa clé A cryptée par la clé K
    pub: { t: 'u8' }, // sa clé RSA publique
    cleAK: { t: 'u8' }, // sa clé RSA privée cryptée par la clé K
    cvA: { t: 'cv' } // sa carte de visite, texte et photocryptée par sa clé A

#### McMemo : Changement des mots clés (hashtags) et mémo attachés à un contact ou groupe 

    id: { t: 'idag' }, // id de l'avatar ou du groupe
    htK: { t: 'u8', n: true }, // hashtags séparés par un espace et crypté par la clé K
    txK: { t: 'u8', n: true } // texte du mémo gzippé et crypté par la clé K

#### ChangementPS : Changement de la phrase secrète de connexion du compte

    hps1: { t: 'ids' }, // hash9 du PBKFD de la phrase secrète réduite du compte.
    hXC: { t: 'ids' }, // hash du PBKFD de la phrase secrète complète
    cleKXC: { t: 'u8' } // clé K cryptée par la phrase secrète

#### NouveauGroupe : Nouveau groupe 
Exception:
- 8001: avatar disparu

    idg: { t: 'idg' }, // id du groupe
    ida: { t: 'ida' }, // id de l'avatar fondateur
    cleAG: { t: 'u8' }, // clé A de l'avatar cryptée par la clé G
    cleGK: { t: 'u8' }, // clé du groupe cryptée par la clé K du compte
    cvG: { t: 'cv' }, // carte de visite du groupe crypté par la clé G du groupe
    msu: { t: 'bool' }, // true si mode simple
    quotas: { t: 'q2' } // {qn, qv} maximum de nombre de notes et de volume fichiers

#### NouveauContact : Nouveau contact 
Exception: 
- 8002: groupe disparu
- 8001: avatar disparu

    idg: { t: 'idg' }, // id du groupe
    ida: { t: 'ida' }, // id de l'avatar contact
    cleAG: { t: 'u8' }, // clé A du contact cryptée par la clé G du groupe
    cleGA: { t: 'u8' } // clé G du groupe cryptée par la clé A du contact

#### ModeSimple: Demande de retour au mode simple d'invitation à un groupe
Exception: 
- 8002: groupe disparu

    idg: { t: 'idg' }, // id du groupe
    ida: { t: 'ida' }, // id de l'avatar demandant le retour au mode simple
    simple: { t: 'bool' } 
    // true 'Je vote pour passer au mode "SIMPLE"'
    // false: 'Annuler les votes et rester en mode UNANIME'

#### AnnulerContact : Annulation du statut de contact d'un groupe par un avatar
Exception: 
- 8002: groupe disparu

    idg: { t: 'idg' }, // id du groupe
    ida: { t: 'ida' }, // id de l'avatar demandant l'annulation.
    ln: { t: 'bool' }  // true Inscription en liste noire

#### InvitationGroupe : Invitation à un groupe'
Exception: 
- 8002: groupe disparu
- 8001: avatar disparu

    idg: { t: 'idg' }, // id du groupe
    idm: { t: 'ida' }, // id du membre invité
    rmsv: { t: 'int', min: 0, max: 4 }, // 0: inviter, 2: modifier, 3: supprimer, 4: voter pour
    flags: { t: 'int', min: 0, max: 255 }, // flags d'invitation
    msgG: { t: 'u8' }, // message de bienvenue crypté par la clé G du groupe
    idi: { t: 'ida', n: true }, // id de l'invitant pour le mode d'invitation simple
    // sinon tous les avatars du comptes animateurs du groupe
    suppr: { t: 'int', min: 0, max: 3 }, // 1-contact, 2:radié, 3-radié + LN
    cleGA: { t: 'u8' } // clé G du groupe cryptée par la clé A de l'invité

#### AcceptInvitation : Acceptation d'une invitation à un groupe
Exception: 
- 8002: groupe disparu
- 8001: avatar disparu

    idg: { t: 'idg' }, // id du groupe
    idm: { t: 'ida' }, // id du membre invité
    iam: { t: 'bool' }, // accepte accès aux membres
    ian: { t: 'bool' }, // accepte l'accès aux notes
    cleGK: { t: 'u8' }, // cle du groupe cryptée par la clé K du compte
    cas: { t: 'int', min: 1, max: 4 }, // 1:accepte 2:contact 3:radié 4:radié + LN
    msgG: { t: 'u8' }, // message de bienvenue crypté par la clé G du groupe
    txK: { t: 'u8', n: true } // texte à attacher à compti/idg s'il n'y en a pas

#### ItemChatgr : Ajout ou effacement d'un item au chat du groupe
Exception: 
- 8002: groupe disparu

    idg: { t: 'idg' }, // id du groupe
    idaut: { t: 'ida', n: true }, // id du membre auteur du texte
    dh: { t: 'dh', n: true }, // date-heure de l'item effacé
    msgG: { t: 'u8', n: true } // texte de l'item

#### MajLectChatgr : mise à jour de la lecture d'un chat du groupe

    idg: { t: 'idg' }, // id du groupe
    lstIm: { t: 'array' } // liste des im des membres ayant lu

#### MajDroitsMembre: Mise à jour des droits d'un membre sur un groupe
Exception: 
- 8002: groupe disparu
- 8001: avatar disparu

    idg: { t: 'idg' }, // id du groupe
    idm: { t: 'ida' }, // id du membre
    nvflags: { t: 'int', min: 0, max: 255 }, // nouveau flags. Peuvent changer DM DN DE AM AN
    anim: { t: 'bool' } // true si animateur

#### RadierMembre : Radiation d'un membre d'un groupe
Exception: 
- 8002: groupe disparu
- 8001: avatar disparu

    idg: { t: 'idg' }, // id du groupe
    idm: { t: 'ida' }, // id du membre
    rad: { t: 'int', min: 1, max: 3 }, // 1-redevient contact, 2-radiation, 3-radiation + ln
    cleGA: { t: 'u8' } // cle G du groupe cryptée par la clé du membre

#### HebGroupe : Gestion de l'hébergement et des quotas d'un grouper
Exception générique:
- 8001: avatar disparu
- 8002: groupe disparu

    idg: { t: 'idg' }, // id du groupe
    nvHeb: { t: 'ida' }, // id de l'avatar nouvel hébergeur
    action: { t: 'int', min: 1, max: 4 }, 
    // 1: 'Je prends l\'hébergement à mon compte',
    // 2: 'Je cesse d\'héberger ce groupe',
    // 3: 'Je transmet l\'hébergement à un autre de mes avatars',
    // 4: 'Je met seulement à jour les nombres de notes et volumes de fichiers maximum attribués au groupe',
    qn: { t: 'int', min: 0 }, // qn: nombre maximum de notes, qv : volume maximum des fichiers
    qv: { t: 'int', min: 0 } // qn: nombre maximum de notes, qv : volume maximum des fichiers

#### SupprAvatar : Suppression d'un avatar
Exception:
- 8001: avatar disparu

    id: { t: 'ida' } // id de l'avatar

#### SupprCompte : Suppression du compte
Exception:
- 8001: avatar disparu

#### NouvelleNote : Création d'une nouvelle note

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ida: { t: 'ida', n: true }, // pour une note de groupe, id de son avatar auteur
    exclu: { t: 'bool', n: true }, // true si l'auteur est exclusif
    pid: { t: 'idag' }, // id de la note parente pour une note rattachée
    pids: { t: 'ids', n: true }, // ids de la note parente pour une note rattachée
    t: { t: 'u8' } // texte crypté

#### RattNote : Gestion du rattachement d'une note à une autre */

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' }, // ids de la note
    pid: { t: 'idag', n: true }, // id de la note parente pour une note rattachée
    pids: { t: 'ids', n: true} // ids de la note parente pour une note rattachée

#### MajNote : Mise à jour du texte d'une note

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' }, // ids de la note
    ida: { t: 'ida', n: true }, // pour une note de groupe, id de son avatar auteur
    t: { t: 'u8' } // texte crypté

#### SupprNote : Suppression d'une note

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' } // ids de la note

#### HTNote : Changement des hashtags attachés à une note par un compte

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' }, // ids de la note
    htK: { t: 'u8', n: true }, // ht personels
    htG: { t: 'u8', n: true }, // hashtags du groupe

#### ExcluNote : Changement de l'attribution de l'exclusivité d'écriture d'une note

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' }, // ids de la note
    ida: { t: 'ida', n: true } // id de l'avatar prenant l'exclusivité

#### GetUrlNf : retourne l'URL de get d'un fichier d'une note
Retour:
- `url` : url de get

#### PutUrlNf : retourne l'URL de put d'un fichier d'une note
Retour:
- `idf` : identifiant du fichier
- `url` : url à passer sur le PUT de son contenu

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' }, // ids de la note
    lg: {t: 'int', min: 0, max: 500000000}, // taille du fichier
    aut: { t: 'ida', n: true }, // pour une note de groupe, id de l'auteur de l'enregistrement
    lidf: { t: 'lidf', n: true } // liste des idf fichiers de la note à supprimer

#### ValiderUpload : validation de l'upload d'un fichier

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' }, // ids de la note
    fic: { t: 'fic' }, // { idf, lg, ficN }
    ida: { t: 'ida', n: true }, // id de l'auteur (pour une note de groupe)
    lidf: { t: 'lidf', n: true } // liste des idf fichiers de la note à supprimer

#### SupprFichier : Suppression d'un fichier d'une note

    id: { t: 'idag' }, // id de la note (avatar ou groupe)
    ids: { t: 'ids' }, // ids de la note
    idf: { t: 'idf' }, // id du fichier à supprimer
    ida: { t: 'ida', n: true } // id de l'auteur (pour une note de groupe)

## Documents
### Les formats _data_ et _compilé_
Que ce soit en SQL ou en Firestore, l'insertion ou la mise à jour d'un document se fait depuis un _objet Javascript_ ou chaque attribut correspond à une propriété de l'objet. 

Par exemple un document `avatars` pour mise à jour en SQL ou Firestore est celui-ci : 

    { id: '3200...00', v: 4, _data_: (Uint8Array) }

Pour avoir le format _data_ une propriété _nom a été ajoutée : 

    { _nom: 'avatars', id ... }

Ce format _data_ est celui utilisé entre session Web et serveur, dans les arguments d'opérations et en synchronisation.

#### Objets _compilé_
Les attributs _data_ contiennent toutes les propriétés sérialisées, celles externalisées `id v ...` et celles internes. 

La fonction `avatar = compile(_data_)` désérialise le contenu de _data_ d'un objet et retourne une instance `avatar` de la classe correspondant au nom (par exemple `Avatars` qui hérite de la classe `GenDoc`) avec tous les attributs sérialisés dans _data_.
- dans le cas de `versions`, _data_ est reconstitué par le provider d'accès à la base avec les seules propriétés externalisées.

La méthode `_data_ = avatar.toData()` reconstitue un objet _data_ sérialisé depuis une instance `avatar`. Elle est utilisée pour exporter une base et est la valeur par défaut de `toShortData()`.

La méthode `avatar.toShortData(op)` est invoquée par l'opération `Sync` et retourne un _data_ dont certains attributs peuvent être omis selon le compte qui a sollicité l'opération `op`.

En session Web le même principe est adopté avec deux différences : 
- `compile()` sur le serveur est **synchrone et générique**.
- en session `async compile()` est écrite pour chaque classe : les méthodes effectuent des opérations de cryptage / décryptage asynchrones et de calculs de propriétés complémentaires spécifiques de chaque classe de document. 

Dans les bases de données, les attributs _data_ sont cryptés par la _clé du site_ fixée par l'administrateur:
- _data_ est décrypté après lecteur de la base,
- _data_ est encrypté avant écriture sur la base.

### Opérations authentifiées de création de compte et connexion
**Remarques:**
- le compte _Administrateur_ n'est pas vraiment un compte mais simplement une autorisation d'appel des opérations qui le concernent lui seul. Le hash de sa phrase secrète est enregistrée dans la configuration du serveur `keys.json app_keys`.
- le compte _Comptable_ de chaque espace est sponsorisé par l'administrateur à la création de l'espace. Ce compte est normal. Sa phrase secrète a été donnée par le comptable à l'acceptation du sponsoring.
- les autres comptes sont créés par _acceptation d'un sponsoring_ qui fixe la phrase secrète du compte qui se créé : après cette opération la session du nouveau compte est normalement active.
- ces opérations sont _authentifiées_ et transmettent les données d'authentification par le _token_ passé en argument porteur du hash de la phrase secrète: dans le cas de l'acceptation d'un sponsoring, la reconnaissance du compte précède de peu sa création effective.

### Opérations authentifiées pour un compte APRÈS sa connexion
Ces opérations permettent à la session Web de récupérer toutes les données du compte afin d'initialiser son état interne.

# Annexe I: Providers Storage
Trois classes _provider_ gèrent le storage, hérite de la classe `GenStProvider` et ont un même interface:
- `FsProvider` : un file-system local du serveur,
- `S3Provider` : un Storage S3 de AWS (éventuellement _minio_).
- `GcProvider` : un Storage Google Cloud Storage.

> En pratique il n'y a pas de raisons à assurer un Storage `s3` sous GAE.

**Constructor**
- `code` : `gc_a fs_a `...
- `site` : lettre du site dans la configuration `keys.json`.

**Méthodes:**

    async ping ()
    getUrl (org, id, idf)
    putUrl (org, id, idf)
    async getFile (org, id, idf)
    async putFile (org, id, idf, data)
    async delFiles (org, id, lidf)
    async delId (org, id)
    async delOrg (org)
    async listFiles (org, id)
    async listIds (org)

Les _paths_ sur Storage sont de la forme `org/id/idf`:
- selon la configuration donnée dans keys.json les données `org` `id` `idf` sont ou non cryptées par la clé du site (et mise en base64 URL)

Le **serveur** gère le storage pour:
- `del...` les suppressions,
  - suppressions individuelles de fichiers,
  - disparition d'un avatar,
  - traitement des _uploads_ perdus.
- `putFile` : pour enregistrer un rapport généré sur le serveur comme l'export CSV des compteurs d'abonnement / consommation.

L'utilitaire `export-st purge-st` utilise les providers,
- pour lister des fichiers (`listFiles listIds`),
- pour supprimer des fichiers (`delOrg`),
- pour transférer des fichiers (`getFile, putFile`).

En conséquence, sauf le cas très particulier des rapports générés sur le serveur, **le serveur n'utilise pas `getFile putFile`**:
- les sessions **accèdent directement** au storage pour upload / download : le contenu des fichiers ne transitent pas par le serveur.
- mais il n'est pas question de transmettre aux sessions les données d'authentification d'accès aux storage pour d'évidentes raisons de sécurité.
- pour permettre à une session de _lire / écrire_ un fichier, un provider génère sur demande du serveur (qui a les accréditations nécessaires) une URL de GET / PUT:
  - elle est fort complexe et contient un jeton d'accès valide pour CE fichier précis pendant une durée limitée.
  - la session emploie juste cette URL sur un simple GET HTTP (download) ou un PUT HTTP (upload).

Ainsi en toute sécurité les sessions échangent des contenus avec le storage (des fichiers attachés aux notes, plus rarement un rapport généré par le serveur), directement et sans que ce contenu ne transite par le serveur.

### Cas particulier du provider File-System
Ce provider ne sert que:
- en test local,
- pour des exports entre deux storage nécessitant un tampon intermédiaire faute de pouvoir techniquement utiliser deux instances du même type de storage (_Google Cloud Storage_ exigeant un seul `projectId`).

Dans ce cas le contenu d'un fichier en GET ou en PUT transite par le serveur puisqu'il est lui-même serveur de storage sur un directory local du host supportant le serveur / utilitaire.

L'URL du serveur `https://.../storage/azer789...` en GET et en PUT est utilisé pour un download / upload:
- l'URL est décodée pour retrouver l'identification du fichier à 3 niveaux `org/id/fichier`,
- le provider courant de storage est sollicité pour effectuer un `getFile / putFile`.
- le contenu du fichier a transité par le serveur avant d'être redirigé vers / depuis le provider courant.

Tout provider doit implémenter `getUrl / putUrl` mais il peut toujours utiliser l'URL générique `https://.../storage/...`
- c'est moins performant puisque le contenu des fichiers va transiter par le serveur,
- ça marche toujours.

Le provider `GcProvider` (_Google Cloud Storage_) propose bien `getUrl / putURL` **MAIS pas en mode _emulator_** où le service est omis. 
- C'est pour cela que l'implémentation de `getUrl / putUrl` utilise l'URL générique et le transfert intermédiaire par le serveur, ce qui n'a aucune importance en test. 
- A noter qu'in fine les fichiers se retrouvent bien dans le storage émulé, ils ont juste fait un transit supplémentaire en mémoire dans le cas d'usage de _emulator_.

# Annexe II : plus d'information sur certains modules dans `src/`

**************EN REVISION********************

## `api.mjs`
Ce module **est strictement le même** que `api.mjs` dans l'application Web afin d'être certain que certaines interprétations sont bien identiques de part et d'autres:
- quelques constantes exportées.
- les classes,
  - `AppExc` : format général d'une exception permettant son affichage en session, en particulier en supportant la traduction des messages.
  - `ID` : les quelques fonctions statiques permettant de déterminer depuis une id si c'est celle d'un avatar, groupe, tribu, ou celle du Comptable.
  - `Compteurs` : calcul et traitement des compteurs statistiques des compteurs statistiques des documents `comptas`.
  - `AMJ` : une date en jour sous la forme d'un entier `aaaammjj`. Cette classe gère les opérations sur ce format.

### `gendoc.mjs`
La classe `GenDoc` représente un document _générique_.

Une classe y est déclarée pour chaque collection / table de documents:
- elle hérite de `GenDoc`
- elle n'est porteuse que de quelques méthodes.

    Espaces Gcvols Tribus Tribu2s COmptas Versions Avataars Groupes
    Notes Transferts Sponsorings Chats Membres

### Fonction `compile (row) -> Objet`
Cette fonction aurait pu être déclarée static de `GenDoc` et a été écrite comme fonction pour raccourcir le texte d'appel très fréquent.

Chaque **row** d'une table SQL ou **document** Firestore apparaît sous deux formes :
- **row** : c'est l'objet Javascript directement stocké en tant que row d'une table SQL ou document Firestore.
- **objet compilé** : c'est une instance d'un des classes de document ci-dessus.

La méthode `compile()` retourne l'objet compilé depuis sa forme row et son nom symbolique : si l'argument row est null, le retour est null (sans levée d'exception).

### Méthode `GenDoc.toRow()`
Réciproquement depuis un objet `a` par exemple de classe `Avatars`, `a.toRow()` retourne sa forme **row** prête à être stockée en row SQL ou document Firestore.

La fonction `compile()` retourne la forme compilée (un objet de sa class spécifique) depuis un row, est dans l'esprit une méthode `statique` de GenDoc mais sous la forme d'une fonction pour commodité d'écriture.

Deux autres fonctions `decryptRow (op, row)` et `prepRow (op, row)` sont utilisés par les providers DB pour crypter / décrypter le _data_ d'un row des classes pour lesquelles le _data_ est crypté par la clé du site.

### Liste restrictive des attributs d'un row
Cette liste est fermée : pour chaque classe la liste exhaustive est donnée.
- `_nom` : nom symbolique de la classe dont est issue le row ('avatar', 'groupe', ...).
- `id` : l'id principale (et unique pour les objets majeurs).
- `ids` : l'id secondaire pour les `Notes Transferts Sponsorings Chats Membres`.
- `v` : sauf Gcvols. Version de l'objet.
- `vcv` : pour Avatars Chats Membres : version de la carte de visite.
- `dlv`: pour Versions Transferts Sponsorings Membres : date limite de validité (`aaaammjj`). A partir de cette date, le document n'est plus _valide_, il est sémantiquement _disparu_. En compilé l'attribut `_zombi` vaut `true`.
- `hps1` : sur Comptas, hash de la phrase secrète raccourcie.
- `dfh` : sur Groupes date de fin d'hébergement.
- `hpc` : sur Avatars, hash de la phrase de contact (pseudo plus ou moins temporaire).
- `_data_` : sérialisation de tous les attributs, dont ceux ci-dessus.

En forme compilé la propriété `_data_` n'est pas elle-même présente mais à la place tous les attributs de la classe sont présents.
- quand _data_ n'existe pas ou est null dans le format row, l'attribut _zombi de la classe correspondante vaut true.

En statique `GenDoc` donne aussi des listes de documents selon leurs modes de gestion afin de faciliter les traitements génériques en particulier d'export et des accès génériques (documents _majeurs-, documents _sous-collection d'un document majeur_) ...

### `modele.mjs`
Ce module comporte trois classes: `Cache Operation AuthSession`.

#### `Cache` : cache des objets majeurs `espaces tribus comptas avatars groupes`

Cet objet gére une mémoire cache des derniers documents demandés dans leur version la plus récente.

Le test pour savoir si la version détenue est la dernière s'effectue dans une transaction et permet de ne pas lire le document de la table ou de la collection si sa version n'est pas plus récente ce qui évite des lectures coûteuses inutiles (et coûteuses monaiterement en Firestore).

Cache gère aussi une mémoire cache de `checkpoint` le document de suivi du GC.

En stockant les document `espaces`, `Cache` fournit également le code de l'organisation d'un espace connu par son ns (son id).

##### `static getRow (op, nom, id)`
Obtient le row de la cache ou va le chercher.
- Si le row actuellement en cache est le plus récent on a évité une lecture effective et la méthode s'est limité à un filtre sur index qui ne coûte rien en FireStore et pas grand chose en SQL.
- Si le row n'était pas en cache ou que la version lue est plus récente IL Y EST MIS:
  - certes la transaction _peut_ échouer, mais au pire on a lu une version, pas forcément la dernière, mais plus récente.

##### `static async getEspaceLazy (op, ns)`
Retourne l'espace depuis celui détenu en cache. C'est seulement s'il a plus de PINGTO minutes d'âge qu'on vérifie sa version et qu'on la recharge le cas échéant.
PAR PRINCIPE, elle est retardée: convient pour checker une restriction éventuelle.

##### `static update (newRows, delRowPaths)`
Utilisée en fin de transaction pour enrichir la cache APRES le commit de la transaction avec tous les rows créés, mis à jour ou accédés (en ayant obtenu la _dernière_ version).

##### `static async getCheckpoint ()`
Retourne le dernier checkpoint enregistré parle GC.

##### `async setCheckpoint (obj)`
Enregistre en base et dans Cache le dernier objet de checkpoint défini par le GC.

##### `static async getEspaceOrg (op, org)`
Retourne le row compilé de l'espace obtenu par son code d'organisation.

##### `static async org (op, id)`
Retourne le code de l'organisation pour un ns donné.

#### `AuthSession`
Cette classe conserve une entrée par session authentifiée et en gère la disparition par défaut d'activité (heartbeat).

#### `Operation`
C'est la classe générique ancêtre des opérations. Chaque opération a une classe spécifique dans le module `operation.mjs` qui hérite de cette classe générique:
- authentification de l'opération,
- enchaînement des phases 1 2 et 3,
- enregistrement effectif des mises à jour en fin de phase-2 (juste avant commit de la transaction),
- signalement des mises à jour au module ws.mjs (en SQL) pour synchronisation WebSocket,
- retour du résultat.

Cette classe propose des méthodes d'interface vers les métodes des providers DB et vers l'accès à Cache: ce ne sont que des commodités syntaxiques.

Enfin cette classe expose aussi une dizaine de méthodes fonctionnelles ayant à être sollicitées depuis plus d'une opération.

_Remarque_: la logique aurait voulu que la classe `Operation` soit incrite dans le module `operations.mjs` : il a été préféré d'isoler le code générique dans un module à part, choix discutable certes mais qui se défend aussi.

Plus de détail en annexe.

# Annexe: détails à propos de la classe `Operation`
Le déclenchement d'une opération `MonOp` sur réception d'une requête `.../op/MonOp`,
- créé un objet `MonOp` héritant de `Operation`,
- invoque successivement :
  - sa méthode `phase1()` qui s'exécute hors de toute transaction, typiquement pour des contrôles d'argments,
  - sa méthode `phase2()` qui s'exécute dans le contexte d'une unique transaction.
  - sa `phase3()` qui s'exécute après le commit de la transaction pour certaines actions de nettoyge et / ou d'accès au storage.

Dans une transaction Firestore aucune lecture n'est autorisée dès qu'une mise à jour a été effectuée. Les mises à jour sont de ce fait _enregistrées et mises en attente_ au cours de la phase 2 et ne seront effectivement faites qu'après la phase 2.

En conséquence, 
- les opérations doivent prendre en compte que la modification d'un document n'est jamais perceptible dans la même transction par une lecture : le cas échéant si nécessaire stocker en mémoire de l'objet opération les mises à jour si elles participent de la logique de l'opération.
- une opération doit veiller à ne pas construire plusieurs mies à jour d'un même document dans des méthodes qui s'ignoreraient.

### Authentification avant `phase1()`
Chaque classe Operation spécifie un attribut authMode qui déclare comment interpréter l'attribut `token` reçu dans l'objet `args` (argments sérialisés reçu dans le body de la requête ou queryString de l'URL). Cet objet est disponible dans `this.args` :
- `authMode === 3` : SANS TOKEN, pings et accès non authentifiés (recherche phrase de sponsoring).
- `authMode === 2` : AVEC TOKEN, créations de compte. Elles ne sont pas encore enregistrées, elles vont justement enregistrer leur authentification.
- `authMode === 1` : AVEC TOKEN, première connexion à un compte : `this.rowComptas` et `this.compta` sont disponibles.
- `authMode undefined` : AVEC TOKEN, cas standard, vérification de l'authentification, voire enregistrement éventuel.

### Méthodes génériques disponibles en phase 1 et 2
`setRes(prop, val)`
- Fixe LA valeur de la propriété 'prop' du résultat (et la retourne).

`addRes(prop, val)`
- AJOUTE la valeur en fin de la propriété Array 'prop' du résultat (et la retourne).

### Méthodes génériques disponibles en phase 2
`insert (row)`
- Inscrit row dans les rows à insérer en phase finale d'écritue, juste après la phase 2.

`update (row)`
- Inscrit row dans les rows à mettre à jour en phase finale d'écritue, juste après la phase2 .

`delete (row) `
- Inscrit row dans les rows à détruire en phase finale d'écritue, juste après la phase 2.
