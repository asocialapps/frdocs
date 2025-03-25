---
layout: page
title: Les déploiements des composants de l'application
---

L'application comporte plusieurs _composants_ (application Web, serveurs / Cloud functions, utilitaires ...), chacun a son processus spécifique de déploiement.

# Déploiement l'application Web
L'objectif est d'obtenir une page Web, `index.html` et les ressources associées, dans un folder à distribuer:
- soit sur un CDN, par exemple **_github pages_: c'est l'option retenue et décrite ci-après**.
- soit dans un serveur Web hébergé,
- soit dans le serveur SRV décrit ci-après quand il héberge les services OP+PUBSUB.

### Configuration
Les paramètres de configuration sont à ajuster dans le fichier `src/config.mjs`. Plusieurs déploiements peuvent avoir exactement la même configuration, typiquement celle par défaut.

Après _build_ on obtient un folder _presque_ prêt à distribuer:
- l'application Web doit disposer des URLs des services OP et PUBSUB.
- celles-ci, qui localisent les services correspondants sur Internet, et donc la base de données et le storage, sont données dans le fichier `public/services.json`.
- un `README.md` est également à fournir.

Voir plus de détails sur ces deux fichiers dans le document [L'application Web](./applicationweb.html), section "Configuration de _runtime_".

> L'URL du serveur (pas spécialement fun) est masquée ici et l'utilisateur ne connaît que l'URL de son application donnée par son _provider_ de l'application comme:

    https://asocialapps.github.io/XX?demo

### Build et test de la build:

    yarn quasar build -m pwa

    # OU
    npm run build:pwa

    # https://github.com/http-party/http-server
    # Installation: npm install -g http-server

    npx http-server dist/pwa -p 8080 --cors -S --cert ../asocial-srv/keys/fullchain.pem --key ../asocial-srv/keys/privkey.pem

    # Plus simplement
    npx http-server dist/pwa -p 8080 --cors

Le résultat est dans `dist/pwa` (environ 40 fichiers pour 5Mo):
- y mettre le fichier `services.json` avec le contenu ci-dessus et `README.md`.

L'application _buildée et configurée_ peut être distribuée depuis `dist/pwa` sur le CDN de son choix, par exemple ci-après dans `github pages`.

#### browser-list pas à jour
Au cours du _build_ un message apparaît souvent en raison de l'obsolescence de la browser-list.

On la met à jour par la commande:

    npx update-browserslist-db@latest

### Distribution par _github pages_

Un _repository_ `github.com` peut aussi gérer un site Web de distribution.
- créer un dépôt `github.com` pour la distribution: par exemple ici `asocialapps` (`https://github.com/asocialapps`).
- créer un repository par distribution à effectuer: par exemple ici `XX`.

Sur le poste qui génère la distribution,
- créer un folder `asocialapps-XX` et y copier les fichiers récupérés du build (dans `./dist/pwa`).
- y ajouter le fichier `services.json`.
- y ajouter un fichier `README.md`qui permet à n'importe qui de reconstituer l'application Web distribuée depuis les sources:
- le dépôt des sources est cité ainsi que la tag qui pointe exactement la révision utilisée.
- la section `patchs` résume ce qui a été modifié dans ces sources avant build. 

Effectuer un `commit + publish` de `XX` vers le dépôt de distribution.

La première fois il faut indiquer dans `github.com` que ce repository doit être diffusé sur `github.io`:
- ouvrir le repository, et sa page de `settings`.
- sélectionner la section `Pages` (à gauche) et fixer les paramètres donnant la branche à distribuer (`master` par défaut).

C'est tout.

Lors de la distribution d'une nouvelle version ultérieure, la configuration de _Pages_ n'a pas à être refaite: le seul fait d'effectuer un `commit + publish` suffira à régénérer le site qui sera accessible par l'URL:

    https://asocialapps.github.io/XX

(C'est bien `.io` pas `.com`).

# Tests externes depuis `localhost` par `ngrok`

`ngrok` permet de créer un _tunnel_ entre _localhost_ et Internet en rendant accessible un serveur HTTP (écoutant le port 8443 par exemple) s'exécutant sur le poste de développement comme s'il était accessible publiquement sur Internet.

Dans sa version gratuite, `ngrok` demande une inscription et permet d'obtenir un _authtoken_.

Sur son compte dans `ngrok` on demande aussi une URL dédiée: celle-ci est générée par `ngrok` et n'est pas au choix. Par exemple:

    exactly-humble-cod.ngrok-free.app

Il faut enregistrer, une fois, son token sur le poste:

    ngrok config add-authtoken MONTOKEN

    >> Authtoken saved to configuration file: /home/daniel/snap/ngrok/179/.config/ngrok/ngrok.yml

Le token est conservé localement, la localisation s'affiche dans le terminal.

Pour ouvrir un _tunnel_, il faut ouvrir un terminal et lancer: 

    ngrok http --domain=exactly-humble-cod.ngrok-free.app 8443

En retour il apparaît l'URL https://... d'accès.

Pour fermer le _tunnel_, interrompre la session en cours dans ce terminal.

Le tunnel établi par Ngrok permet d'utiliser cette URL depuis n'importe où, en particulier depuis un mobile, pour joindre le serveur s'exécutant sur le `localhost` du poste de développement.

# Déploiements des services OP et PUBSUB

## _Obfuscation_ des données sensibles
### Certificats du serveur
Si le serveur n'est pas géré par un provider externe (Google, AWS, Azure ...), il est à démarrer comme une simple application `node` à qui il faut fournir les certificats https `fullchain.pem` et `privkey.pem`.
- ces données **NE DOIVENT PAS** être exposées dans `git`. Elles sont stockées dans le folder `./keys` qui DOIT figurer en `.gitignore`.
- le build des services **NE CONTIENT PAS** ce folder car les certificats doivent être renouvelés assez souvent et indépendamment du build d'une nouvelle version des services.
- c'est sur le host de production directement que le folder `./keys` est installé, avec en général un script de renouvellement automatique des certificats.

### Tokens divers
Ces tokens sont des autorisations d'usage d'API, des authentifications diverses. Ils sont inscrits comme des entrées majeures dans le fichier `./keys.json`. Les entrées actuelles sont:
- `app_keys` : les clés de cryptage des _sites_, le mot de passe de l'administrateur technique (en fait son PBKFD) et les clés `vapid` de notification _web push_.
- `service_account` : l'authentification d'un compte Google.
- `s3_config` : l'authentification du fournisseur d'accès au _storage_ S3.

Ce fichier NE DOIT PAS être exposé dans git, il figure dans `.gitignore` (d'ailleurs dans le cas contraire github par exemple envoie des alertes).

Mais ces données doivent être intégrées dans la _build_ des services:
- il n'est pas souhaitable que cette _build_ les fasse apparaître en clair. L'hébergeur qui installe les services n'a pas à les voir passer explicitement dans les fichiers de _build_.

Le fichier `./keys.json` est _obfusqué_ par la commande:

    node src/gensecret.mjs

qui en génère le fichier `./src/secret.mjs`:
- le fichier `./keys.json` est parsé, l'objet Javascript résultant est sérialisé et crypté (par un couple clé / salt généré dans le code) et encodé en base64.
- le résultat est écrit dans `./src/secret.mjs`

A l'initialisation des services, l'import de ce fichier expose l'encodage en base64 généré. Ce texte est converti en binaire, décrypté et l'objet correspondant disponible dans la configuration interne.

> Le fichier `./src/secret.mjs` **N'EST PAS** communiqué à `git`. Il est a régénérer à chaque fois que `keys.json` change (et à la première installation).

> Certes la lecture des sources permet de comprendre comment l’obfuscation a été réalisée. Mais le _hacking_ en est compliqué pour l'hébergeur des services qui doit écrire du code pour l'extraire et avoir lu le code de la build pour en saisir le procédé et les clés. 

## Typologie des déploiements possibles

Selon les configurations choisies, on peut effectuer des _build_ et déployer les services OP et PUBSUB selon plusieurs options. Ci-après la liste des options documentées, et celles a priori non pertinentes avec la raison associée.

### Déploiements pour un site NON géré par un provider externe
Un serveur NON géré est un serveur dont on assure soi-même la configuration et la surveillance d'exploitation. Typiquement:
- un site de production sur une ou des VMs hébergés chez un fournisseur standard.
- un serveur de test _personnel_ pour une exploitation de démonstration ou de test, en ayant un nom de domaine spécifique.

Sur un site non géré, il faut installer:
- _éventuellement_ un `nginx` (ou équivalent) capable d'effectuer un _load balancing_ entre plusieurs instances de serveur HTTP assurant un service OP afin d'accroître la puissance disponible.
- _éventuellement_ une base de données, `Sqlite` (ou `Postgresql`), locale, et en gérer la sécurité / backup / restore. Mais il est aussi possible d'utiliser un service d'hébergement `Firestore` (ou par extension DynamoDB -Amazon-, ou CosmoDB -Microsoft Azure-) où un service d'hébergement externe `Postgresql`.

Les déploiements documentés sont les suivants:
- **Serveur OP**: serveur assurant le seul service OP.
- **Serveur PUBSUB**: serveur assurant le seul service PUBSUB.
- **Serveur SRV**: serveur assurant les deux services OP+PUBSUB.

### Déploiement pour un site GAE géré
Google App Engine (GAE) est une solution pour déployer un _serveur_ assurant les mêmes fonctionnalités qu'un site NON géré avec les remarques suivantes.

**GAE est une _ferme_ de serveurs:** une ou plusieurs instances peuvent s'exécuter en parallèle, le nombre pouvant est borné à UNE instance. Au bout d'un certain temps sans réception de requête, l'instance est arrêtée et sera relancée à l'arrivée d'une nouvelle requête:
- c'est exactement le même comportement qu'un _Cloud Function_, si ce n'est que la durée de vie en l'absence de requête est plus long (une heure au lieu de 5 minutes pour fixer les idées).

**Déployer le service OP _seul_ sur GAE n'a pas d'intérêt a priori:** plutôt utiliser un Cloud Function.

**Déployer le service PUSUB _seul_ sur GAE n'a pas d'intérêt a priori:** plutôt utiliser un Cloud Function.

**Déployer les deux services OP+PUBSUB sur GAE à un intérêt** de simplification d'administration:
- la comparaison des coûts avec un mix de Cloud Functions n'a pas été faite.
- il faut borner le nombre d'instances à UNE, PUBSUB ne peut pas être multi-instances: en conséquence c'est **une option de _faible_ puissance**.
- le mot _faible_ est flou: le débit potentiel peut cependant être suffisant dans les cas d'usage par des organisations de taille modeste ou moyenne.
- dans les cas de faible trafic, le coût tombe en dessous du minimum facturable, l'hébergement devenant gratuit.

GAE _pourrait_ assurer la distribution de l'application Web (et du site documentaire statique) mais **ce n'est pas une bonne idée**:
- ça obligerait à refaire un déploiement de l'ensemble même quand seule l'application Web a changé en provoquant une interruption certes faible de disponibilité.
- ça présente aux utilisateurs une URL d'accès (celle de l'application Web) assez abscons et où Google apparaît.

**Il reste préférable d'assurer séparément la distribution de l'application par `github pages` (ou autre)**. Ceci permet aussi de changer le déploiement des services OP et PUBSUB pour des solutions différentes meilleures en termes de coûts / performances de manière transparente pour les utilisateurs, ce qui est souhaitable.

### Déploiement par des Cloud Functions (CF)
**Le service OP peut être déployé par CF**, sans contrainte sur le nombre d'instances en parallèle.

**Le service PUBSUB peut être déployé par CF**, avec contrainte d'UNE instance au plus.

Il n'est documenté ici que l'usage de _Google Cloud Functions_: les deux autres options chez AWS et Azure sont à tester et documenter, même si en théorie les adaptations de code à effectuer semblent marginales.

# Choix de la _base de données_

### Autogestion de la base de données
C'est possible pour les _providers_ `SQLite` et `PostgreSQL`.

C'est une gestion lourde et humainement contraignante pour une organisation.

### Service hébergé de la _base de données_
#### Firestore
Service géré par Google. Si le débit est faible on peut tomber sous le seuil de facturation.

En pratique ceci impose _naturellement_ à opter pour _Google Storage_ pour le _storage_ des fichiers.

### PostgreSQL
Il y a plusieurs offres sur le marché avec un coût d'entrée minimal d'une vingtaine d'euros mensuels: certes le débit va être important, mais c'est une solution à réserver si le coût de _Firestore_ devient prohibitif.

### DynamoDB (AWS) et CosmoDB (Azure)
Ces classes _providers_ n'ont pas été écrites:
- sur le papier il n'y a pas de contradictions avec les contraintes à respecter pour l'interface _provider_.
- les autres _providers_ ont environ 500 lignes de code: le temps à passer en compréhension / test des API, puis en création des comptes Amazon ou Azure, puis en tests, est plus important que l'effort d'écriture à proprement parlé du code.

# Choix du _storage_

### Autogestion
Réservé à un site de test avec le provider _file system_.

### Google Storage
A choisir si les options GAE et CF chez Google sont prises.

### Amazon S3
S3 est un _interface_, le _provider_ a été testé avec `minio`.
- S3 est bien entendu disponible chez AWS.
- d'autres fournisseurs existent sur le marché, la comparaison des coûts n'a pas été faite.
- gérer soi-même S3 avec `minio` n'est pas vraiment réaliste en production.

Le Storage de Azure serait à écrire: environ 500 lignes de code.

# Fichiers de configuration

### Fichier `keys.json`
Ce fichier contient des autorisations et tokens spécifiques de l'administrateur du site:
- il est exclu de git afin de ne pas exposer ces données au public,
- sa sécurité est gérée par l'administrateur technique par ses propres moyens.

> L'outil `node src/gensecret.mjs` génère un fichier _obfusqué_ `src/secret.mjs`.

Il comporte les constantes suivantes:
- `app_keys`: voir ci-après.
- `service_account` : pour les déploiements sur Google Cloud Platform. Procédure décrite en annexe.
- `s3_config` : le token d'authentification au service S3 choisi. Procédure décrite en annexe.

      {
      "vapid_private_key": "...",
      "vapid_public_key": "...",

      "app_keys" : {
        "admin":  ["..."],
        "sites":  {
          "A": { "k": "...", "db": [true, true, true], "st": [true, true, true] },
          "B": { "k": "...", "db": [false, false, false], "st": [false, false, false] }
        }
      },

      "service_account" : { ... },

      "s3_config" : { ... },

      "alertes" : {
        "_url": "http://.../script.php",
        "_pwd": "...",

        "_sendgrid_api_keyX": "...",
        "_from": "daniel@sportes.fr"
      }
      }

#### `vapid_private_key` et `vapid_public_key`
Ce couple de clés permet au service PUBSUB de pousser des notifications vers l'application Web qui DOIT avoir dans sa configuration la `vapid_public_key`.
- génération par l'outil _vapid_: `node src/tools.mjs vapid`
- la clé publique est transmise dans la configuration de / des applications ayant accès à ce serveur / services.

#### `app_keys`
`admin`: donne la ou les hash des phrases secrètes de l'administrateur technique.
- lancer l'application Web (n'importe laquelle)
- ouvrir le panneau d'outils: barre du bas, icône _engrenage_,
- onglet _Tester une phrase secrète_
- saisir une phrase secrète par exemple `les framboises sont bleues cette année`,
- copier le texte qui apparaît dans `SHA256 du PBKFD de la phrase complète`.

    ZgHxc7BeHgR7z5HLpidsMd4XmSJmfCCulgu7cfwB9V8=

Ceci donne une clé à mettre dans les propriétés de `app_keys`:

Le login s'effectuera ainsi:

    Organisation: `admin`
    Phrase secrète: `les framboises sont bleues cette année`

`sites` : cette rubrique permet d'enregistrer la clé et les options de cryptage des _sites_:
- habituellement un administrateur n'est concerné que par _son_ site, de code `A` typiquement.
- toutefois s'il doit réaliser des exports de base de données pour d'autres sites, il doit disposer de la clé de cryptage du site cible, d'où dans ce cas l'existence d'un site `B`.

L'attribut `k` est obtenu comme la propriété admin:
- les propriétés _data_ des tables / documents sont toujours cryptées par cette clé.

L'attribut `db` donne trois booléens indiquant si la base crypte aussi,
- [0] : le **code de l'organisation** préfixant les `id` et propriétés `hk`;
- [1] : les ids principales des documents;
- [2] : les ids secondaires des documents.

L'attribut `st` donne trois booléens indiquant si le _storage_ crypte,
- [0] : le **code de l'organisation**;
- [1] : les ids des avatars / groupes;
- [2] : les ids des fichiers.

#### `service_account`
Jeton d'accès au _Google account_ Dans le cas de déploiement sur Google (GAE / Firestore / Google Cloud Storage).

#### `s3_config`
Jeton d'accès au storage d'interface S3.

#### `alertes`
Certaines alertes graves peuvent donner lieu à envoi d'un email à une adresse (typiquement celle de l'administrateur technique) afin qu'il soit notifié au plus vite de la nécessité de consulter le log du serveur.

Les propriétés sont celles d'un serveur externe d'envoi de mail mis en place par l'administrateur technique:
- le couple `_url _pwd` est relatif à un serveur Web,
  - dont l'url est donnée ici,
  - et dont un mot de passe est donné ici.
  - Voir en annexe l'API que doit implémenter un tel serveur.
- le couple `_sendgrid_api_key _from` sont deux propriétés pour solliciter le service sendGrid (Twilio). Usage expérimental en évaluation. Dans le cas d'usage d'autres services du marché, leur configuration serait données ici.


### Folder `keys`
**Ce folder contient le certificat du domaine de l'application** tels que générés par `Lets Encrypt` (https://letsencrypt.org/) par exemple:
- `fullchain.pem`
- `privkey.pem`

Ce folder,
- est exclu de git pour ne pas exposer les clés du domaine au public,
- est **externe** au _build_ afin que sur le site on puisse renouveler ces clés (obsolètes régulièrement) sans refaire un déploiement.

**Ce folder ne sert qu'aux déploiements _serveur NON géré_** et est inutile en cas de déploiement GAE et CF (dont les certificats sont gérés par le fournisseur).

### Fichier `src/config.mjs`
- Il est inclus dans git.
- Chaque déploiement a le sien.
- Il comporte,
  - une partie _fonctionnelle_ qui peut avoir une valeur commune pour tous les déploiements du site.
  - une partie _technique_ spécifique de chaque déploiement et détaillée ci-après.

Il comporte des lignes donnant les _configurations_ des _providers_ de DB et de Storage:
- pouvant être cités dans la section `run`,
- pouvant être cité sur les lignes de commande: `node src/tools.mjs ...`

      // Configuration nommées des providers db et storage
      s3_a: { bucket: 'asocial' },
      fs_a: { rootpath: './fsstoragea' },
      fs_b: { rootpath: './fsstorageb' },
      gc_a: { bucket: 'asocial-test1.appspot.com' },
      sqlite_a: { path: './sqlite/testa.db3' },
      sqlite_b: { path: './sqlite/testb.db3' },
      firestore_a: { },

#### Configuration des _providers_
Chaque nom de configuration comporte: le `nom du provider`, `_`, `lettre d'identification`. Les noms sont:
- Base données:
  - `sqlite`: accès à une base de données SQLite. Options:
    - `{ path: '...' }` Path du fichier `.db3` de la base données.
  - `firestore`: accès à Google Firestore. Options: `{ }`
- Storage:
  - `fs`: accès à un folder du File-System local. Options:
    - `{ rootpath: '...' }` Path de la racine.
  - `gc`: accès au Google Cloud Storage. Options:
    - `{ bucket: '...' }` identifiant du bucket réservé à cet usage.
  - `s3`: accès à un Storage Amazon S3. Options:
    - `{ bucket: '...' }` identifiant du bucket réservé à cet usage.

#### La section env: {...}
_Certains_ paramètres (typiquement ceux de _l'emulator de Google Cloud Platform) **doivent** figurer en variables d'environnement. On les déclare dans la section env.
Cas identifiés:
- `STORAGE_EMULATOR_HOST`: pour EMULATOR de Google Storage
- `FIRESTORE_EMULATOR_HOST`: pour EMULATOR de Google Firestore

      env: {
        STORAGE_EMULATOR_HOST: 'http://127.0.0.1:9199', // 'http://' est REQUIS
        FIRESTORE_EMULATOR_HOST: 'localhost:8085' // Remplace le port par défaut 8080
      }

Toutes les entrées de cette section seront converties en _variable d'environnement_.

# Déploiements _Serveur_

Les déploiements possibles sont:
- déploiement d'un service OP.
- déploiement d'un service PUBSUB.
- déploiement _SRV_, incluant OP et PUBSUB.

Chaque déploiement demande:
- d'ajuster la configuration pour chaque cas à déployer,
- d'effectuer un _build_,
- de distribuer le résultat du _build_ sur le site de production.

# Déploiements _Serveur_ d'un service OP (sans PUBSUB)

### Fichier `src/keys.json`
- les clés `vapid...` sont inutiles (mais ne nuisent pas).

`service_account`
- uniquement si le _provider_ de base de données est Firestore.

`s3_config`
- uniquement si le _provider_ de Storage est S3.

alertes
- uniquement si les alertes graves sont poussées sur le mail de l'administrateur technique.

### Folder `keys`
Le certificat du domaine est requis pour les serveurs NON gérés.

### Fichier `src/config.mjs`

    export const config = {
      // Paramètres fonctionnels
      allocComptable: [8, 2, 8],
      allocPrimitive: [256, 256, 256],
      heuregc: [3, 30], // Heure du jour des tâches GC
      retrytache: 60, // Nombre de minutes avant le retry d'une tâche

      // Configuration du déploiement
      env: { },

      fs_a: { rootpath: './fsstoragea' },
      gc_a: { bucket: 'asocial-test1.appspot.com' */ },

      sqlite_a: { path: './sqlite/test.db3' },
      firestore_a: { },

      pathlogs: './logs',
      pathkeys: './keys',

      run: {
        site: 'A',
        origins: new Set(['http://localhost:8080']),
        nom: 'test asocial-sql', // pour le ping
        pubsubURL: 'https://test.sportes.fr/pubsub/',
        mode: 'https',
        port: 8443,
        storage_provider: 'gc_a',
        db_provider: 'sqlite_a',
        projectId: 'asocial-test1',

        rooturl: 'http://test.sportes.fr:8443'
      }
    }

**Commentaires**
- `gc_a:` configuration du provider Google Cloud Storage. Par commodité on peut en décrire plusieurs (gc_a gc_b etc.)
- `sqlite_a:` configuration du provider DB SQLite. Par Par commodité on peut en décrire plusieurs (sqlite_a sqlite_b etc.)
- `firestore_a:` configuration du provider DB Firestore. Par Par commodité on peut en décrire plusieurs (firestore_a firestore_b etc.)
- `run.site: 'A'` indique l'entrée de `app_keys.sites` qui détient la clé de cryptage du site.
- `run.origins:` Set des `origin` des sites de CDN délivrant l'application Web. Si vide ou que la directive est absente, pas de contrôle sur l'origine.
- `run.nom:` sert uniquement à l'affichage lors d'une requête ping.
- `run.mode:` 'https' (par exception en test 'http').
- `run.port:` numéro de port d'écoute.
- `pubsubURL`: URL où le service OP va trouver le service PUBSUB,
- `run.storage_provider:` identifiant du provider de Storage (référencé au-dessus).
- `run.db_provider:` identifiant du provider de DB (référencé au-dessus).
- `run.projectId:` ID du project Google si l'un des providers storage / db est un service de Google.
- `run.rooturl:` en général absent. URL externe d'appel du serveur qui ne sert qu'à un provider de storage qui doit utiliser le serveur pour délivrer une URL get / put file. Cas storageFS et storageGC en mode _emulator_.

### Build par `webpack`

Fichier `webpack.config.mjs` :

    import path from 'path'
    export default {
      entry: './src/server.js', 
      target: 'node',
      mode: 'production',
      output: {
        filename: 'op.js',
        path: path.resolve('dist/op')
      }
    }

Le _résultat du _build_ par webpack ira dans le folder `dist/op` et son .js principal y sera `op.js`

#### Gestion de `package.json` et `build`
En développement il y a une ligne:

    "type": "module",

Pour effectuer un _build_ **il faut renommer cette directive** "typeX" (ou la supprimer).

Commande dans un terminal (à la racine du projet):
- `npx webpack`
- durée de 30s à 1 minute: être patient, rien ne s'affiche avant la fin.

**Il faut renommer la directive ""typeX" en "type"** sinon le développement ne marche plus.

Dans le folder `dist/op` insérer un fichier `package.json` ne contenant que `{}`:

    echo "{}" > dist/op/package.json

Si on veut exécuter immédiatement le résultat du _build_ directement dans le folder `dist/op`, `node` va chercher un `package.json`, y compris au niveau de folder supérieur et va trouver le `package.json` du projet qui a une directive `"type"module"` ce qui met l'exécution en erreur car `op.js` utilise un `require`.
La présence du `package.json` _fake_ permet d'éviter ce problème.

### Exécution de test dans `dist/op`
- si le _provider_ de storage était par exemple `sqlite_a: { path: './sqlite/testa.db3' },` vérifier qu'il y a bien une base `dist/op/sqlite/testa.db3`.
- si le _provider_ de storage était par exemple `fs_a: { rootpath: './fsstoragea' },` dans `config.mjs`, vérifier qu'il existe bien un folder `dist/op/fsstorage`.

Exécution:

    cd dist/op
    node op.js

> **Attention:** `op.js` est le _vrai_ exécutable et écrit vraiment dans la _vraie_ base et le _vrai_ storage.

# Déploiements _Serveur_ d'un service PUBSUB (sans OP)

### Fichier `src/keys.mjs`
`app_keys`
- les clés `vapid...` SONT REQUISES.

`service_account`
- inutile (mais ne nuit pas).

`s3_config`
- inutile (mais ne nuit pas).

### Folder `keys`
Le certificat du domaine est requis.

### Fichier `src/config.mjs`

    export const config = {
      // Paramètres fonctionnels inutiles mais ne nuisent pas

      // Configuration du déploiement
      env: { },

      pathlogs: './logs',
      pathkeys: './keys',

      run: {
        site: 'A',
        nom: 'test asocial-pubsub', // pour le ping
        mode: 'https',
        port: 8444
      }
    }

**Commentaires**
- `run.site: 'A'` indique l'entrée de `app_keys.sites` qui détient la clé de cryptage du site
- `run.origins:` Set des origin des sites de CDN délivrant l'application Web et des sites OP. Si vide ou que la directive est absente, pas de contrôle sur l'origin.
- `run.nom:` sert uniquement à l'affichage lors d'une requête ping.
- `run.mode:` 'https' (par exception en test 'http').
- `run.port:` numéro de port d'écoute. Attention à ne pas donner le même numéro de port que celui du service OP si c'est un serveur sur le même host.

### Build par `webpack`

Fichier `webpack.config.mjs` :

    import path from 'path'
    export default {
      entry: './src/pubsub.js', 
      target: 'node',
      mode: 'production',
      output: {
        filename: 'pubsub.js',
        path: path.resolve('dist/pubsub')
      }
    }

Le _résultat du _build_ par webpack ira dans le folder `dist/pubsub` et son .js principal y sera `pubsub.js`

#### Gestion de `package.json` et `build`
En développement il y a une ligne:

    "type": "module",

Pour effectuer un _build_ **il faut renommer cette directive** "typeX" (ou la supprimer).

Commande dans un terminal (à la racine du projet):
- `npx webpack`
- durée de 30s à 1 minute: être patient, rien ne s'"affiche avant la fin.

**Il faut renommer la directive ""typeX" en "type"** sinon le développement ne marche plus.

Dans le folder `dist/pubsub` insérer un fichier package.json ne contenant que `{}`:

    echo "{}" > dist/pubsub/package.json

Si on veut exécuter immédiatement le résultat du _build_ directement dans le folder `dist/pusub`, `node` va chercher un `package.json`, y compris au niveau de folder supérieur et va trouver le `package.json` du projet qui a une directive `"type"module"` ce qui met l'exécution en erreur car `pubsub.js` utilise un `require`.
La présence du `package.json` _fake_ permet d'éviter ce problème.

### Exécution de test dans `dist/pubsub`

Exécution:

    cd dist/pubsub
    node pusub.js

> **Attention:** `pubsub.js` est le _vrai_ exécutable et pousse de _vraies_ notifications.

# Déploiements _Serveur_ d'un service OP+PUBSUB

### Fichier `src/keys.mjs`
`app_keys`
- les clés `vapid...` sont REQUISES.

`service_account`
- uniquement si le _provider_ de base de données est firestore.

`s3_config`
- uniquement si le _provider_ de Storage est S3.

### Folder `keys`
Le certificat du domaine est requis.

### Fichier `src/config.mjs`

    export const config = {
      // Paramètres fonctionnels
      allocComptable: [8, 2, 8],
      allocPrimitive: [256, 256, 256],
      heuregc: [3, 30], // Heure du jour des tâches GC
      retrytache: 60, // Nombre de minutes avant le retry d'une tâche

      // Configuration du déploiement
      env: { },

      fs_a: { rootpath: './fsstorage' },
      gc_a: { bucket: 'asocial-test1.appspot.com' */ },

      sqlite_a: { path: './sqlite/testa.db3' },
      firestore_a: { },

      pathlogs: './logs',
      pathkeys: './keys',

      run: {
        site: 'A',
        origins: new Set(['http://localhost:8080']),
        nom: 'test asocial-sql', // pour le ping
        pubsubURL: null,
        mode: 'https',
        port: 8443,
        storage_provider: 'gc_a',
        db_provider: 'sqlite_a',
        projectId: 'asocial-test1',

        rooturl: 'http://test.sportes.fr:8443'
      }
    }

**Commentaires**
- `gc_a:` configuration du provider Google Cloud Storage. Par commodité on peut en décrire plusieurs (gc_a gc_b etc.)
- `sqlite_a:` configuration du provider DB SQLite. Par Par commodité on peut en décrire plusieurs (sqlite_a sqlite_b etc.)
- `firestore_a:` configuration du provider DB Firestore. Par Par commodité on peut en décrire plusieurs (firestore_a firestore_b etc.)
- `run.site: 'A'` indique l'entrée de app_keys.sites qui détient la clé de cryptage du site
- `run.origins:` Set des origin des sites de CDN délivrant l'application Web. Si vide ou que la directive est absente, pas de contrôle sur l'origin.
- `run.nom:` sert uniquement à l'affichage lors d'une requête ping.
- `pubsubURL`: null, le serveur assure localement ce service aussi.
- `run.mode:` 'https' (par exception en test 'http').
- `run.port:` numéro de port d'écoute.
- `run.storage_provider:` identifiant du provider de Storage (référencé au-dessus).
- `run.db_provider:` identifiant du provider de DB (référencé au-dessus).
- `run.projectId:` ID du project Google si l'un des providers storage / db est un service de Google.
- `run.rooturl:` en général absent. URL externe d'appel du serveur qui ne sert qu'à un provider de storage qui doit utiliser le serveur pour délivrer une URL get / put file. Cas storageFS / storageGC en mode _emulator_.

### Build par `webpack`

Fichier `webpack.config.mjs` :

    import path from 'path'
    export default {
      entry: './src/server.js', 
      target: 'node',
      mode: 'production',
      output: {
        filename: 'srv.js',
        path: path.resolve('dist/srv')
      }
    }

Le _résultat du _build_ par webpack ira dans le folder `dist/srv` et son .js principal y sera `srv.js`

#### Gestion de `package.json` et `build`
En développement il y a une ligne:

    "type": "module",

Pour effectuer un _build_ **il faut renommer cette directive** "typeX" (ou la supprimer).

Commande dans un terminal (à la racine du projet):
- `npx webpack`
- durée de 30s à 1 minute: être patient, rien ne s'affiche avant la fin.

**Il faut renommer la directive ""typeX" en "type"** sinon le développement ne marche plus.

Dans le folder `dist/srv` insérer un fichier package.json ne contenant que `{}`:

    echo "{}" > dist/srv/package.json

Si on veut exécuter immédiatement le résultat du _build_ directement dans le folder `dist/srv`, `node` va chercher un `package.json`, y compris au niveau de folder supérieur et va trouver le `package.json` du projet qui a une directive `"type"module"` ce qui met l'exécution en erreur car `srv.js` utilise des `require`.
La présence du `package.json` _fake_ permet d'éviter ce problème.

### Exécution de test dans `dist/srv`
- si le _provider_ de storage était par exemple `sqlite_a: { path: './sqlite/testa.db3' },` vérifier qu'il y a bien une base `dist/op/sqlite/testa.db3`.
- si le _provider_ de storage était par exemple `fs_a: { rootpath: './fsstoragea' },` dans config.mjs, vérifier qu'il existe bien un folder `dist/op/fsstoragea`.

Exécution:

    cd dist/srv
    node srv.js

> **Attention:** `srv.js` est le _vrai_ exécutable et écrit vraiment dans la _vraie_ base et le _vrai_ storage.

# Déploiements de  _tools_
### Fichier `src/keys.mjs`
`keys`
- les clés vapid... sont inutiles (mais ne nuisent pas).

`service_account`
- uniquement si l'un des _providers_ de base de données est Firestore.

`s3_config`
- uniquement si l'un des _providers_ de Storage est S3.

### Folder `keys`
Non utilisé.

### Fichier `src/config.mjs`

    export const config = {
      // Configuration du déploiement
      env: { },

      fs_a: { rootpath: './fsstoragea' },
      gc_a: { bucket: 'asocial-test1.appspot.com' */ },

      sqlite_a: { path: './sqlite/testa.db3' },
      firestore_a: { },

      run: {
      }
    }

**Commentaires**
- `gc_a:` configuration du provider Google Cloud Storage. Par commodité on peut en décrire plusieurs (gc_a gc_b etc.)
- `sqlite_a:` configuration du provider DB SQLite. Par Par commodité on peut en décrire plusieurs (sqlite_a sqlite_b etc.)
- `firestore_a:` configuration du provider DB Firestore. Par Par commodité on peut en décrire plusieurs (firestore_a firestore_b etc.)

### Build par `webpack`

Fichier `webpack.config.mjs` :

    import path from 'path'
    export default {
      entry: './src/tools.mjs', 
      target: 'node',
      mode: 'production',
      output: {
        filename: 'tools.js',
        path: path.resolve('dist/tools')
      }
    }

Le _résultat du _build_ par webpack ira dans le folder `dist/tools` et son .js principal y sera `tools.js`

#### Gestion de `package.json` et `build`
En développement il y a une ligne:

    "type": "module",

Pour effectuer un _build_ **il faut renommer cette directive** "typeX" (ou la supprimer).

Commande dans un terminal (à la racine du projet):
- `npx webpack`
- durée de 30s à 1 minute: être patient, rien ne s'affiche avant la fin.

**Il faut renommer la directive ""typeX" en "type"** sinon le développement ne marche plus.

Dans le folder `dist/tools` insérer un fichier package.json ne contenant que `{}`:

    echo "{}" > dist/tools/package.json

### Exécution de test dans `dist/tools`
Les providers qui seront cités dans la ligne de commande doivent être déclarés.

Exécution:

    cd dist/tools
    node tools.js ... arguments

Voir en annexe les arguments de `tools`.

> **Attention:** `tools.js` est le _vrai_ exécutable et écrit vraiment dans la _vraie_ base et le _vrai_ storage.

# Déploiement _GAE_ 
Il faut avoir configuré son projet Google pour:
- activer App Engine
- utiliser Firestore: créer la base.
- utiliser Storage: créer son bucket.
- activer CRON sur App Engine.

Le déploiement se fait dans un directory dédié, qui n'a pas lieu d'être archivé dans git.

> Il n'a pas été possible d'effectuer un build webpack avant déploiement: en conséquence le déploiement s'effectue en _source_.

Les fichiers spécifiques au déploiement GAE sont à préparer dans `asocial-srv/gae`.
- Copie et adaptation des fichiers sous `asocial-srv`:
  - `keys.json`
    - supprimer l'entrée `s3_config`
  - `config.mjs`
    - première ligne: `EMULATOR = false`
    - vérifier dans le `run` : http/port db_provider storage_provider.
  - `package.json`
    - enlever les sections `devDependencies packageManger`
    - vérifier qu'il y a bien `type: module`
    - vérifier qu'il n'y a qu'un seul `script: {"start": "node ./src/},secret.js"}`.
- Fichiers spécifique de `gae`:
  - `app.yaml`: vérifier la version de node.
  - `cron.yaml`: ajuster l'heure si nécessaire.
  - `.cloudignore`
  - `depl.sh`. Ce fichier recopie les fichiers nécessaires dans le directory de déploiement.

## Scénario de déploiement

#### Préparer le folder de déploiement: `asocial-gae1` (par exemple)
Y créer les folders `src` et `node_modules`.

      cd ./asocial-srv/gae
      ./depl.sh

      cd ../../asocial-gae1
      npm install     
      # OUI npm, pas yarn, deploy utilise package-lock.json

      node src/gensecret.mjs 
      # pour intégrer une éventuelle mise à jour de keys.json

`npm install` à deux fonctions:
- permettre d'effectuer un test final après déploiement,
- générer un package-lock.json qui accélère le déploiement ET fixe les versions exactes des modules.

#### Tester localement
On peut tester le serveur avec: node src/server.js

**MAIS ça s'exécuterait sur la base de production**, c'est inopportun. Il faut donc:
- changer dans `config.mjs` la première ligne `EMULATOR = true`
- lancer l'emulator dans une autre fenêtre et l'initialiser avec des données de test d'intégration.

Après tests, changer à nouveau dans `config.mjs` la première ligne `EMULATOR = false`.

## Déployer depuis `asocial-gae1`

    gcloud app deploy --verbosity debug --no-cache

no-cache : sinon plantage du build step 2 en cherchant à comparer avec une version antérieure.

Quelques minutes ..., puis si nécessaire (si `cron.yaml` a changé par rapport à l'opérationnel):

    gcloud app deploy cron.yaml

C'est rapide.

Dans un autre terminal `gcloud app logs tail` permet de voir les logs de l'application quand ils vont survenir.

Les logs complets s'obtienne depuis la console Google du projet (menu hamburger en haut à gauche `>>> Logs >>> Logs Explorer`).

# Déploiements _Cloud Function_ des services OP et PUBSUB
A rédiger, après un premier déploiement réel.

# L'utilitaire `upload`

Un _browser_ ne peut pas écrire dans le _file-system_ de son poste. L'application Web offre la possibilité du _download_ d'une sélection de notes et de leurs fichiers attachés. Pour ce faire elle invoque l'URL http://localhost:33666

upload est un micro serveur Web qui, une fois lancé, écoute ce port: il reçoit des requêtes PUT émises par l'application Web, une par _fichier_ à écrire localement, et en écrit le contenu sur le répertoire local:
- **le path du fichier 'abcd...'** en relatif au directory courant d'exécution, est donné dans l'URL en base64 URL: http://localhost:33666/abcd...
- le contenu du fichier est dans le body de la requête.

Un _build + packaging_ délivre deux exécutables, un pour Linux `upload`, l'autre Windows `upload.exe`, autonomes: ils embarquent un runtime `node.js` qui dispense l'utilisateur d'une installation un peu technique de `node.js`.

Il est aussi possible de mettre à disposition upload.js: dans ce cas le PC devra installer node et lancer l'exécution par `node upload.js`.

Les fichiers envoyés par PUT sont installés dans le répertoire courant ou s'exécute `upload`. Argument optionnel: numéro de port d'écoute.

    cd ...
    upload 33666
    upload

### Build
Depuis le folder où a été installé `upload` depuis git:

    npm run build
    npx webpack // devrait aussi fonctionner.

### Packaging
_Installation de pkg_

    npm install -g pkg

_Génération des exécutables_

    cd dist 
    pkg -t node14-win upload.js
    pkg -t node14-linux upload.js

Créé des exécutables pour linux `upload` et windows (en x64) `upload.exe`.

### Distribution
Dans un dépôt de distribution (par exemple `github.com/asocialapps`):
- créer un repository `upload`

Localement publier dans ce repository:
- `index.html`
- `upload.exe`
- `upload`
- `upload.js`

Dans le settings de ce repository, rubrique Pages, donner `main` comme branche de publication.

    <!DOCTYPE html>
    <html>
      <head>
        <title>Utilitaires "upload"</title>
      </head>
      <body>
        <div><a href="./upload.exe">Pour Windows (x86): upload.exe</a></div>
        <div><a href="./upload">Pour Linux (x86): upload</a></div>
      </body>
    </html>

Cette page permet de télécharger les exécutables. 

On _pourrait_ être un peu plus bavard dans `index.html` et y inscrire un minimum de manuel d'utilisation.

# Annexe I: CLI `tools`

`tools` est invoqué depuis son folder d'installation.

    node tools.js commande arguments

> `tools` a été _buildé_ avec un fichier de configuration `src/config.mjs` qui en général comporte des _paths_ relatifs ou absolus. S'assurer de la validité de ceux-ci depuis le folder d'exécution de `tools`.

Les commandes sont:
- `export-db`: exporter un _espace_ d'une base de données sur un autre _espace_ d'une autre base de données.
- `export-st`: exporter un _espace_ d'un Storage sur un autre _espace_ d'un autre Storage.
- `purge-db`: purge d'un _espace_ d'une base de données.
- `vapid`: génération d'un nouveau couple de clés privée / publique VAPID. Pas d'arguments, résultat dans `./vapid.json`.

### `export-db -s --in ... --out ...`
- `-s` : optionnel. Simulation, rien n'est écrit.
- `--in N,org,prov_x,S` - Espace _source_
  - `N`: lettre / chiffre de l'espace `0..9 a..z A..Z`
  - `org`: code de l'organisation
  - `prov`: nom du provider: `sqlite firestore`
  - `x`: le provider `prov_x` doit être décrit dans la configuration.
  - `S`: lettre du site dans la liste des sites de la configuration.
- `--out N,org,prov_x,S` - Espace _cible_

### `purge-db --in ...`
- `--in N,org,prov_x,S` - Espace à purger
  - `N`: lettre / chiffre de l'espace `0..9 a..z A..Z`
  - `org`: code de l'organisation
  - `prov`: nom du provider: `sqlite firestore`
  - `x`: le provider `prov_x` doit être décrit dans la configuration.
  - `S`: lettre du site dans la liste des sites de la configuration.

### `export-st -s --in ... --out ...`
- `-s` : optionnel. Simulation, rien n'est écrit.
- `--in org,prov_x` - Storage _source_
  - `org`: code de l'organisation
  - `prov`: nom du provider: `fs gc s3`
  - `x`: le provider `prov_x` doit être décrit dans la configuration.
- `--out org,prov_x` - Cible

### Exemples:

    node tools export-db --in 1,doda,sqlite_a,A --out 2,coltes,sqlite_b,B
    node tools export-db --in 1,doda,sqlite_a,A --out 1,doda,firestore_a,A
    node tools export-db --in A,doda,firestore_a,A --out A,doda,sqlite_b,A

    Exemple export-st:
    node tools export-st --in doda,fs_a --out doda,gc_a

    Exemple purge-db
    node tools purge-db --in 2,coltes,firebase_b,A
    node tools purge-db --in 2,coltes,sqlite_b,B

# Utilitaire d'envoi de mails d'alertes

Certaines alertes peuvent être notifiées sur le mail de l'administrateur technique.

Un des moyens est d'utiliser un site Web d'URL `http://monsite.truc.fr/` ayant PHP et acceptant le mailing par PHP. Y inscrire un script `sendMail.php` ayant l'interface suivant:
- il accepte des requêtes POST `application/x-www-form-urlencoded` avec les arguments suivants:
  - `mailer`: 'A' # Au cas où le script accepte plusieurs configuration de mail.
  - `mdp`: # mot de passe du service SMTP utilisé
  - `to`: # adresse email de l'administrateur technique
  - `subject`: # texte du sujet
  - `text`: # Corps du texte
- cas de succès d'envoi du mail retourne `OK: date-heure` et en cas d'échec `KO: date-heure`.

Un exemple figure dans `etc/sendMail.php` et un exemple de soumission de requête est dans `etc/testMail.html`.
