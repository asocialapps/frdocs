---
layout: page
title: L'application Web
---

L'application est une page Web, _PWA: progressive web app_:
- elle n'a qu'une page (au sens Web),
- un script _service worker_ assure la disponibilité de la page en l'absence de réseau,
- l'application **reçoit** des notifications web-push transmises par le browser qui les a reçues du service PUBSUB afin de notifier une évolution des documents du périmètre du compte connecté.
- l'application **émet** des requêtes HTTP:
  - vers le service OP, opérations de mises à jour / consultations / synchronisation,
  - vers le _Storage_ pour upload / download de fichiers attachés aux notes.

Elle est structurée,
- selon le framework `vuejs.org`,
- et selon `quasar.dev` en tant que surcouche de `vuejs`.

L'architecture de l'application sépare:
- sa **mémoire** globale contenant:
  - des constantes de configuration,
  - les documents du compte courant, _non réactives_ et _réactives_.
- ses **vues** ayant chacune trois parties:
  - `template` : description déclarative HTML+ de ce qui apparaît à l'écran,
  - `script` : variables affichables dans le template, variables de calcul local à la vue, et fonctions sur ces variables.
  - `style` SASS : classes de style utilisées dans le template de la vue. 

# Configuration de l'application

## Configuration _custom buildée_
Quand elle est est adaptée, ce qui n'est pas du tout obligatoire, elle est _buildée_ avec l'application et ne peut pas être modifiée après _build_.

#### Dans le script `src/app/config.mjs`
Elle peut tout à fait être conservée par défaut, sauf la propriété `BUILD` qui donne la date-heure du build de l'application.

#### Dans les scripts `src/i18n/fr-FR/index.js`
Les scripts `fr-FR` et `en-EN` donnent les _traductions_ en français et en anglais.

Une customisation est certes possible mais à gérer avec délicatesse: il faut concaténer le script _par défaut_ avec un script de _surcharges ponctuelles_.

#### Dans l'aide en ligne `src/assets/help/...`
L'aide en ligne peut aussi être customisée mais mais à gérer avec délicatesse: les fichiers d'aide _par défaut_ étant à fusionner avec des fichiers de _surcharges ponctuelles_.

> Un hébergeur procédant à une _customisation_ doit rendre public une liste de fichiers ayant patché la customisation par défaut et le script de _patch_ associé: n'importe qui sera ainsi en mesure de refaire le _build_ correspondant en s'assurant que les sources _open source_ de départ n'ont pas été altérés par des patchs compromettant la confidentialité.

## Configuration de _runtime_
L'application une fois _buildée_ (distribuable `[distrib]/`) est un folder comportant peu de fichiers:
- elle peut être utilisée par tous les hébergeurs ayant cette même customisation de _build_.
- un seul ficher `[distrib]/etc/urls.json` est spécifique de chaque hébergeur.

    {
    "opurl" : "http://localhost:8443",
    "pubsuburl" : "http://localhost:8443",
    "docsurls": { "fr-FR": "http://localhost:4000/fr", "en-EN": "http://localhost:4000/en" },
    "vapid_public_key": "BC8J60JGGoZRHWJDrSbRih-0qi4Ug0LPbYsnft668oH56hqApUR0piwzZ_fsr0qGrkbOYSJ0lX1hPRTawQE88Ew",
    }

_**Remarques:**_
- ce fichier _peut_ être téléchargé / affiché par n'importe qui: il ne comporte aucune information confidentielle.
- ce fichier ne comporte _que_ les informations propres à chaque hébergeur, qui chacun a modifié la distribution sortie de _build_ pour y mettre ses propres URLs.
- `opurl`: URL du service OP de l'hébergeur.
- `pubsuburl`: URL du service PUBSUB de l'hébergeur (_peut_ être égal à celui de OP).
- `docsurls`: URLs dans les différentes langues supportées du site de documentation de l'application. Chaque hébergeur _peut_ proposer son propre site de documentation OU choisir celui standard.
- `vapid_public_key`: chaque hébergeur _peut_ avoir généré son propre couple de clés VAPID (pour le service web-push), mais peut aussi opter par facilité pour le couple de clés générées pour le test de l'application.

> Les fichiers de l'application _buildée_ sont, à la limite lisible _en clair_ (du moins à peu près), par n'importe qui. C'est un moyen pour déterminer si elle a subi des transformations depuis le source public _open source_.

## Chargement de la configuration: `config-store.js`
Il est assuré par le script `src/boot/appconfig.mjs` qui est lancé au boot de l'application, avant affichage de _la_ vue racine `App.vue`.

Par commodité, la configuration _compilée_ est stockée dans le _store_ `src/stores/config-store.js`: c'est peu rationnel, la configuration par principe ne devrait pas changer après chargement initial et n'aurait pas besoin d'être disponible dans un _store_ réactif. Toutefois, par commodité et parce que ce sont les seules données permanentes d'une session du browser, 5 propriétés sont gérées par le _service-worker_ et évoluent donc (rarement) en cours de session: en particulier la propriété `nouvelleVersion` (effectivement réactive) est affichée dans `App.vue`.

Ce _store_ est le seul qui est constant depuis le chargement de l'application et **ne subit pas de _reset_** à la connexion à un nouveau compte.

    nouvelleVersion: false, // passe à true quand une nouvelle version est disponible
    registration: null, // objet de registration du SW
    subJSON: '???', // subscription obtenu de SW sérialisé
    pageSessionId: '', // rnd, identifiant universel du chargement de la page (session browser)
    nc: 0, // numéro d'ordre de connexion dans la session

Ces données sont fondamentales pour la gestion des notifications web-push:
- `subJSON` est le token web-push obtenu par le _service-worker_ depuis le browser.
- `pageSessionId` est un identifiant aléatoire représentant la session de l'application.
- `nc` est un numéro d'ordre de 1 à N de connexion croissant dans une session.

> Le couple `pageSessionId, nc` identifie universellement une session de connexion à un compte dans un browser dont l'exécution est universellement identifiée par `subJSON`.

# `service-worker` et `web-push`
`quasar` gère le _service-worker_ et met à disposition les scripts de customisation suivants:
- `src/pwa/register-service-worker.js` : ce script est invoqué à l'enregistrement du service-worker, donc _dans_ l'application.
- `src/pwa/custom-service-worker.js` : ce script est une extension du service-worker et s'exécute donc dans le SW lui-même.

### `src/pwa/register-service-worker.js`
Le script propose des débranchements lors des évènements:
- `ready` : à cette occasion l'objet `registration` est transmis pour stockage dans `config-store`.
- `updatefound updated` : cet événement active une action de `config-store`. La propriété (réactive) `nouvelleVersion` passe à `true` sur réception de `updated` (`updatefound` est ignoré), ce qui provoque l'affichage dans `App.vue` du bouton `Nouvelle version disponible`. 

> Mis à part l'affichage d'une alerte quand une nouvelle version est disponible, l'application ne gère pas son remplacement effectif effectué automatiquement par `quasar`.

### Recevoir les notifications web-push
Il faut d'abord indiquer au browser que l'application est à l'écoute de celles-ci:
- quand le SW est `ready` il transmet sur l'évènement qui indique cet état un objet `registration` qui représente l'enregistrement du SW. Cet objet est conservé dans `config-store.regisration`.
- on obtient le jeton du browser depuis cet objet `registration`:

    let subscription = await this.registration.pushManager.getSubscription() // déjà faite
    if (!subscription) subscription = await this.registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: b64ToU8(stores.config.vapid_public_key)
      })
    this.subJSON = JSON.stringify(subscription)

Mais pour cela il faut fournir la clé publique VAPID correspondante, celle dont la clé privée est employée par le service PUBSUB pour émettre les notifications web-push.

La clé `subJSON` spécifique de cette session du browser est conservée dans `config-store` et sera communiquée aux service OP et PUBSUB pour leur permettre d'émettre des notifications web-push à cette session du browser (identifiée universellement sur la planète).

### `src/pwa/custom-service-worker.js`
Ce script étend le script de SW et donne la possibilité d'écouter les évènements _push_ transmis par le browser à toutes les sessions à l'écoute de ceux-ci.
- il faut répercuter ces évènements à l'application alors qu'ils ont été reçus par le SW.
- on utilise pour ça le service `BroadcastChannel` initialisé pour traiter le channel `channel-pubsub` et permet cette communication événementielle entre SW et application.
- la `payload` des évènements reçus y est envoyée par un `postMessage`.

    const broadcast = new BroadcastChannel('channel-pubsub')

    self.addEventListener('push', (event) => {
      const payload = event.data ? event.data.text() : ''
      broadcast.postMessage({ type: 'pubsub', payload: payload})
    })

### Dans l'application `src/boot/appconfig.js`
Au boot de la session, une fonction `msgPush` est déclarée à titre de _callback_ sur réception des messages postés par `postMessage` dans le SW:

    new BroadcastChannel('channel-pubsub').onmessage = msgPush
    ...
    async function msgPush (event) {
      if (event.data && event.data.type === 'pubsub') {
        try {
          const obj = decode(b64ToU8(event.data.payload))
          if (obj.sessionId === stores.session.sessionId)
            syncQueue.synchro(obj.trLog)
        } catch (e) {
          console.log(e.toString())
        }
      }
    }

La fonction `msgPush`,
- ignore les payloads dont le type n'est pas `pubsub`,
- ignore les payloads relatives à des web-push concernant une session qui n'est pas la session courante (vieux messages parvenus tardivement).
- invoque la méthode `synchro` de l'objet `syncqueue` en charge de traiter les notifications web-push émises par le service PUBSUB.

> La partie _technique_ de réception et filtrage des notifications web-push est donc assez simple et concentrée sur une vingtaine de lignes de code, certes distribuées dans 4 scripts différents.

# Connexions successives
L'application étant lancée, elle peut vivre une succession de _sessions connectées_:
- une session connectée commence par un _reset_ de toutes les données de l'application SAUF `config-store` qui est donc le seul conteneur de données permanent dans la session de l'application:
- l'état d'une session connectée est mémorisé dans `session-store`:
  - la propriété `status` indique son état majeur:
    - 0: fermée / pas encore ouverte (phase de login), 
    - 1: en chargement, login effectuée mais la session n'est pas encore opérationnelle,
    - 2: session opérationnelle,
    - 3: session opérationnelle MAIS admin (pas une _vraie_ session).
  - dès le status 1, l'identification de la session connectée `sessionId` est connue.
- quelques autres données _immuables_ de la session (restant constantes durant la session):
  - `mode`: synchronisé, incognito, avion
  - `org`: code l'organisation,
  - `authToken`: token d'authentification pour le service OP,
  - `phrase`: phrase secrète (du moins ses hash),
  - `nombase`: nom de la base IDB locale pour une session synchronisée ou avion.
- les autres données dépendent de l'état courant de la session, même si comme `clek` elles n'évoluent plus une fois fixées. 

Une session sort de l'état _fermée_ par une opération de `connexion` et retourne à l'état `fermée` par une opération de `deconnexion`.

# Données d'une session connectée
Toutes les données sont effacées au début d'une session connectée, il ne reste rien des sessions antérieures (à l'exception de `config-store`).

Toutes ces données sont obtenues depuis deux sources:
- **à la connexion depuis les documents lus de la base IDB** pour des sessions  ou avion.
- **depuis les documents rapportés par l'opération `Sync`**.

Dans les deux cas, les documents sont,
- _compilés_ depuis leur forme sérialisée en leur forme instance de leur classe (`src/modele.mjs`), 
- accumulés dans un _buffer_, une instance de la classe `SB` (`src/synchro.mjs`),
- puis en une seule fois, sans interruption, mémorisés dans les _stores_,
- in fine en une seule transaction stockés dans IDB (si nécessaire).

L'ensemble des _stores_ présente en conséquence toujours un état _cohérent_ (atomique) représentant exactement un état _passé_ (de peu) mais consistent des documents du périmètre du compte.

> Les données des _stores_ ne sont JAMAIS mises à jour directement depuis les vues ou les opérations et reflètent toujours le résultat de la dernière opération `Sync` (`src/synchro.mjs`) effectuée.

## Données NON réactives
Ces données sont immuables après initialisation, plus exactement d'éventuelles réinitialisations donneraient les mêmes valeurs. Elles sont stockées dans de simple `Maps`.

### Registre des clés des avatars (classe : `RegCles`  - `src/modele.mjs`)
Le registre des clés délivre la clé d'un avatar depuis son id et est alimenté à chaque rencontre d'une clé d'avatar dans la compilation des documents. Comme un avatar ne change pas de clé, cette information n'a pas besoin d'être réactive.

### Registre des clés des _chats_ et _clés privées_ des avatars (classe : `RegCc`  - `src/modele.mjs`)
La clé privée d'un avatar se découvre à la compilation d'un avatar (ou du compte), sa clé immuable par principe est stockée dans ce registre.

La clé d'un _chat_ est décryptée, soit par la clé K du compte, soit par la clé RSA de l'avatar: ce registre permet d'obtenir la clé d'un _chat_ quelque soit la façon dont elle a été cryptée.

## Données _réactives_, les _stores_ : `src/stores/...`
`./stores.mjs`
- répertoire des stores existant avec un getter par store.

### Données issues des _documents_ du périmètre du compte
`./session-store.js`
- tous les documents singletons du compte: `compte compta compti espace partition`
- les id _courantes_ `avatatarId` pour l'avatar courant, etc.
- les getters sur les états du compte.

`./avatar-store.js`
- une entrée par avatar donnant aussi ses `chats sponsorings tickets notes` (juste l'ids et le volume de fichier).

`./groupe-store.js`
- une entrée par groupe donnant son `chatgr`, ses `membres notes` (juste l'ids et le volume de fichier)
- les `invits` du compte.

`./note-store.js`
- donne l'arbre des notes tel qu'affiché avec un node par note et par avatar / compte.

`./people-store.js`
- une entrée par _contact_ avec la liste des groupes dont il est membre et des avatars avec lesquels il a un chat.

### Autres données
`./ficav-store.js`
- une entrée par fichier accessible en mode avion, avec son statut de chargement.

`./pp-store.js`
- une entrée par note ou fichier du presse-papier. 

### Données UI
`./ui-store.js`
- des données reflétant l'état UI de la session.
- page courante / précédente.
- stack des _dialogues_ actuellement ouverts.
- phrase secrète en cours de saisie.
- pages d'aide empilées.
- etc.

`./filtre-store.js`
- état courant des critères de filtres de sélection pour les pages ayant un filtrage.

# Fichiers de l'application
`/public` : ces fichiers se retrouvent tels quels dans la distribution
  - `./etc/urls.json` : valeur utilisées en test.
  - `./icons/favicon-128x128.png`
  - `./favicon.ico`

`/src` : les _sources_ de l'application

`/src-pwa` : les fichiers pour le service-worker
  - custom-service-worker.js
  - register-service-worker.js

Fichiers de configuration technique _customisés_
- `.gitignore`
- `.yarnrc.yml`
- `jsconfig.json`
- `package.json` : voir annexe.
- `quasar-config.js` : voir annexe.

## Le folder `src/app`
`api.mjs`
- fichier strictement identique en application et services OP / PUBSUB.
- ID et clés, génération etc.
- classes utilitaires devant avoir exactement le même comportement.

`base64.mjs`
- de binaire à base64 et réciproquement. Identique à celui utilisé en service OP / PUBSUB.

`config.mjs`
- élément de customisation de l'application, constantes de configuration.

`db.mjs`
- gère la base IDB, la lecture des tables et les écritures synchronisées.

`modele.mjs`
- registres _non réactifs_ des clés.
- toutes les classes de _document_ de l'application et leur compilation depuis le format _row_ sérialisé reçu par _Sync_ et lu depuis IDB.

`net.mjs`
- fonctions d'accès à Internet GET / POST, aux services OP et PUBSUB et au _storage_.

`operation.mjs`
- classe abstraite ancêtre des _opérations_.

`opeartions4.mjs`
- une classe par opération.

`synchro.mjs`
- classe `Queue` : reçoit les notifications web-push, les met en queue et déclenche les opérations `Sync` correspondantes de remise à niveau des documents du périmètre.
- classe `SB` : buffer accumulant les documents à ranger en _store_ en une fois et à ranger en IDB en une transaction.
- fonction de `connexion` / `deconnexion`
- l'opération `Sync` : récupération du delta des documents mis à jour sur le serveur depuis le dernier Sync.
- quelques opérations un peu _spéciales_: `AcceptationSponsoring GetEspace GetSynthese GetPartition EchoTexte getPub ErreurFonc PingDB RefusSponsoring ProlongationSponsoring ExistePhrase GetSponsoring GetCompta GetComptaQV GetNotifC GetEspaces`

`util.mjs`
- diverses fonctions utilitaires.

`webcrypto.mjs`
- fonctions utilisant la cryptographie (Web-Crypto).

## Le folder `src/assets`
`./fonts`
- les fontes utilisées dans l'application

`./help`
- les ressources de l'aide en ligne.

`./*.png *.jpg *.svg *.bin`
- quelques images et deux sons `beep.bin` `cliccamera.bin`

## Le folder `src/boot`
Scripts exécutés au boot:
- `appconfig.js` : chargement de la configuration.
- `axios.js i18n.js` : pour quasar.

## Le folder `src/css`
- fichiers de style `css` et `sass`, définissant des valeurs par défaut et des customisations à importer sur demande dans les sections `<style>` des vues.

## Le folder `src/i18n`
Un sous folder par langue traduite dont le fichier `index.js` exporte tous les termes traduits.

## Le folder `src/router`
Inutilisé, à laisser tel quel.

## Les "vues"
`src/App.vue`
- La vue racine de l'application, LE layout de LA page de l'application.

`src/pages`
- les vues de type _page_.

`src/panels`
- les vues de type _panel_.

`src/dialogues`
- les vues de type _dialogues simples_.

`src/components`
-les vues de composants importés dans les précédents.

# Application

Une première partie décrit la structure visuelle de l'application en App et pages, panels / dalogues / components.

La seconde partie décrit les données qui sont affichées, le _modèle_ et les données le cas échéant persistentes localement.

La trosième partie décrit les opérations de connexion, synchronisation et les autres opérations d'échanges avec le serveur.

# Structure visuelle
**App** est LE layout unique décrivant LA page de l'application. Elle est constituée des éléments suivants:
- **headaer**
  - _boutons à gauche_: aide, notifications, menu, accueil.
  - _titre de la page courante_. Le cas échant une seconde barre affiche les onglets pour les pages ayant des onglets.
  - _boutons à droite_: fichiers visibles en avion, presse-papier, ouverture du drawer de recherche.
- **footer**:
  - _boutons à gauche_: aide, langue, mode clair / foncé, outils, statut de la session,
  - _information du compte_ connecté et son organisation,
  - _bouton de déconnexion_.
- **drawer de filtre à droite** affichant les filtres de recherche pour les pages en ayant. 
  - il s'affiche par appui sur le bouton de recherche (tout en haut à droite).
  - quand la page est assez large, le drawer de filtre reste affiché à côté de la page principale, sinon sur la page principale qui en est partiellement recouverte.
- **container de la page principale**: 
  - il contient à un instant donné une des pages listées dans la section "Pages". 
  - celles-ci sont formées par un tag `<q-page>` qui s'intègre dans le tag `<q-page-container>` de App.

**App inclut quelques dialogues singletons** afin d'éviter leurs inclusions trop multiples:
- ces dialogues n'ont pas de propriétés, c'est le contexte courant qui fixe ce qu'ils doivent afficher.
- chaque dialogue dans App est gardée par un `v-if` de la variable modèle qui l'ouvre.
- `DialogueErreur DialogueHelp PressePapier PanelPeople PanelMembre OutilsTests ApercuCv PhraseSecrete`

**App a quelques dialogues internes simples:**
- _ui.d.aunmessage_ : Gestion d'un message s'affichant en bas
- _ui.d.diag_ : Affiche d'un message demandant confirmation 'j'ai lu'
- _ui.d.confirmFerm_ : demande de confirmation d'une fermeture de dialogue avec perte de saisie
- _ui.d.dialoguedrc_ : choix de déconnexion. Déconnexion, reconnexion, continuer

La logique embarquée se limite à:
- détecter le changement de largeur de la page pour faire gérer correctement l'ouverture du drawer de filtre dans stores.ui,
- gérer le titre des pages,
- se débrancher vers les pages demandée.

### Pages, panels, dialogues, components
Chaque page au sens ci-dessus peut importer des _panels, dialogues, components_.

#### Panels
Ce sont dialogues qui s'affichent sur la gauche en pleine hauteur avec une largeur qui peut être `'sm md lg'`.
- ils se ferment par appui sur le chevron gauche en haut à gauche.

#### Dialogues
Ce sont des dialogues qui s'affichent sur une partie de la page avec une largeur `'sm md lg'` et pas en pleine hauteur.
- ils se ferment par appui sur la croix en haut à gauche.

#### Components
Ils sont simplment incrustés dans le flow d'affichage de la page / panel / dailogue qui les importent.

#### Maîtrise des cycles d'importation
Un tel cycle est une faute de conception que Webpack détecte (il ne sait pas comment faire) mais n'indique malheureusement pas clairement.

Pour chaque composant un numéro de couche est géré:
- les composants n'important rien ont pour numéro de couche 1.
- tous composants a pour numéro de couche le plus des numéros de couche des composants importés + 1.
- la feuille Excem Dépendances.xls en tien attachement ce qui ermet de s'assurer qu'un cycle n'a pas fortuitement été introduit dans la conception.

## Styles clairs et foncés
Le style global peut être clair ou foncé selon la variable de Quasar `$q.dark.isActive`

Quand il y a des listes à afficher, il est souhaiatble d'afficher une ligne sur deux avec un fond légérement différent, donc avec un style dépendant de l'index `idx` de l'item dans la liste qui le contient. D'où les classes suivantes:
- `sombre sombre0`: fond très sombre, fonte blanche pour les idx pairs (ou absents).
- `sombre1`: fond un peu moins sombre, fonte blanche pour les idx impairs.
- `clair clair0`: fond très clair, fonte noire pour les idx pairs (ou absents).
- `clair1`: fond un peu moins clair, fonte noire pour les idx impairs.

Dans util.js les fonctions suivantes fixent dynamiquement le fond à appliquer selon que le mode sombre est activé ou non et l'index idx éventuel:
- `dkli (idx)` : fond _dark_ ou _light_ selon idx
- `sty ()` : fond _dark_ ou _light_
- `styp (size)` : pour un dialogue, fond _dark_ ou _light_, largeur fixée par size (`'sm' 'md' 'lg'`) et ombre claire ou foncée.

### L'affichage MarkDown par VueShowdown
Le component VueShowdown affiche le contenu d'un texte MD dans un `<div>`. 

Sa classe de style principal `markdown-body` porte un nom **fixe** de manière assez contraignante (ne supporte pas un nom de class dynamique). Ceci oblige à un avoir un component distinct pour chaque style désiré:
- `SdBlanc [texte]`: la fonte du texte est blanche (pour des fonds foncés).
- `SdNoir [texte]`: la fonte du texte est noire (pour des fonds clairs).
- `SdRouge [texte]`: la fonte du texte est rouge (pour des fonds clairs ou foncés).

Dans ces components le fond N'EST PAS fixé, il est transparent, et suivra celui de l'environnement. Mais celui de la fonte doit l'être, d'où le component suivant:
- `SdNb [texte idx]`: il choisit entre SdBlanc et SdNoir selon, a) que le mode Quasar _dark_ est actif ou non, b) que idx passé en propriété est absent ou pair ou impair.

> Remarque: de facto seul SdNb est utilisé dans les autres éléments. Et encore car l'affichage d'un MD s'effectue quasiment tout le temps par le component ShowHtml qui englobe SdNb. Toutefois il existe des cas ponctuels d'utilisation de SdBlanc et SdRouge.

### Les mots clés
Les mots clés sont attachés:
- à des contacts ou des groupes connus du compte par **McMemo**,
- à des notes par **NoteMc**.

Ils sont affichés par **ApercuMotscles** qui permet d'éditer le choix par **ChoixMotscles**.

### Gestion des dialogues
Les objectifs de cette gestion sont:
- de gérer une pile des dialogiues ouverts et en conséquence de pouvoir les fermer sans avoir à se rappeler de leur empilement éventuel,
- pouvoir fermer tous les dialogues ouverts à l'occasion d'un changement de page ou d'une déconnexion.

Du fait qu'général un composant _peut_ être instancié plus d'une fois, les modèles gérant ses dialogues doivent être différenciés pour chaque instance.

Au `setup` chaque instance de composant reçoit un identifiant unique attribué par le store `ui`:

    const ui = stores.ui
    const idc = ui.getIdc(); onUnmounted(() => ui.closeVue(idc))

Toutefois quelques dialogues gérés par `App.vue` ont pour `idc` la valeur `'a'`.

L'ouverture d'un dialogue interne `d1` à un composant s'effectue par:

    ui.oD('d1', idc) // idc ou 'a' pour les dialogues de App.vue

Le fermeture du dernier dialogue ouvert s'effectue par `ui.fD()`

Le dialogue `d1` est déclaré par:

    <q-dialog v-model="ui.d[idc].d1" persistent>

    // ou pour un dialogue de niveau App
    <q-dialog v-model="ui.d.a.d1" persistent>

**Remarques:**
- `onUnmounted(() => ui.closeVue(idc))` permet de nettoyer les variables inutiles en sortie d'une vue.
- au changement d'une page, tous les dialogues ouverts sont fermés.
- c'est aussi le cas lors de la déconnexion.



## Components

### Les filtres
La page principale App a un drawze à droite réservé à afficher les filtres de sélection propres à chaque page et permettant de restreindre la liste des éléments à afficher dans la page (par exemple les notes).

Chaque filtre est un component simple de saisie d'une seule donnée: la valeur filtrée étant stockée en store.

- **FiltreNom**: saisie d'un texte filtrant le début d'un nom ou un texte .contenu dans un string.
- **FiltreMc**: liste de mots clés (qui peuvent être soit requis, soit interdits).
- **FiltreNbj**: saise d'un nombre jours 1, 7, 30, 90, 9999.
- **FiltreAvecgr**: case à cocher 'Membre d\'un groupe' pour filtre des contacts.
- **FiltreTribu**: menu de sélection de:
  - '(ignorer)',
  - 'Compte de ma tranche de quotas',
  - 'Sponsor de ma tranche de quotas',
- **FiltreAvecsp**: case à cocher 'Comptes "sponsor" seulement'.
- **FiltreNotif**: menu de sélection de la gravité d'une notification:
  - '(ignorer)',
  - 'normale ou importante',
  - 'importante'
- **FiltreRac**: menu de sélection d'unstatut de chat:  
  - '(tous, actifs et raccrochés)',
  - 'Chats actifs seulement',
  - 'Chats raccrochés seulement'
- **FiltreSansheb**: case à cocher 'Groupes sans hébergement'.
- **FiltreEnexcedent**: case à cocher 'Groupes en excédent de volume'.
- **FiltreAinvits**: case à cocher 'Groupes ayant des invitations en cours'.
- **FiltreStmb**: menu de sélection du statut majeur d'un membre.
  - '(n\'importe lequel)',
  - 'contact',
  - 'invité',
  - 'actif',
  - 'animateur',
  - 'DISPARU',
- **FiltreAmbno**: filtre des membres d'un groupe selon leurs drots d'accès:
  - '(indifférent)',
  - 'aux membres seulement',
  - 'aux notes seulement',
  - 'aux membres et aux notes',
  - 'ni aux membres ni aux notes',
  - 'aux notes en écriture'
- **FiltreVols**: menu permettant de sélectionner un volume de fichiers d'une note 1Mo, 19Mo, 100,Mo 1Go
- **FiltreTri**: sélectionne un crtière de tri dans une des deux listes TRIespace et TRItranche définies au dictionnaire. 
  - sur tranche: stores.avatar.ptLcFT tri selon l'une des 9 propriétés des tranches listées en tête de stores.avatar.
  - sur espace: PageEspace effectue un tri selon 17 propriétés des synthèses.
 
### Les boutons
Ils n'importent aucune autre vue et sont des "span" destinés à figurer au milieu de textes.
- **BoutonHelp**: ouvre une page d'aide.
- **BoutonLangue**: affiche la langue courante et permet de la changer.
- **NotifIcon**: affiche le statut de notification de restriction et ouvre PageCompta. 
- **BoutonMembre**: affiche le libelleé d'un membre, son statut majeur et optionnellment un bouton ouvrant un PanelMembre qui le détaille. N'est importé que dans ApercuGroupe.
- **BoutonBulle** (3): affiche en bulle sur clic, un texte MD figurant dans le dictionnaire des traductions.
- **BoutonBulle2** (3): affiche en bulle sur clic, un texte MD qui a été composé dynamiquement en respectant les traductions.
- **BoutonUndo**: affiche une icône undo, disable ou non selon la condition passée en propriété.
- **BoutonConfirm**: active la foncion de confirmation quand l'utilisateur a frappé le code aléatoire de 1 à 255 qui lui est proposé.
- **QueueIcon**: petit rond de couleur au-dessus d'une icöne pour marquer l'existence d'une queue de fichiers en téléchargement.

### ChoixQuotas (1)
Saisie des quotas d'abonnement / consommation à affecter à une tranche, un compte, un groupe.

### TuileCnv (1)
Affiche dans une tranche ou un espace le taux d'occupation.

### TuileNotif (1)
Affiche dans une tranche ou un espace les notifications.

### MenuAccueil (1)
Affiche le menu principal aussi inclus dans la page d'accueil.

### ApercuMotscles (3)
Liste les mots clés sur une ligne. 
- ouverture du choix des mots clés sur bouton d'édition.

Import: ChoixMotscles

Dialogue:
- AMmcedit: édition / zoom des mots clés

### EditeurMd (3)
Editeur de texte en syntaxe MD, visible soit en HTML soit en texte pur.
- zoom en plein écran possible,
- insertion d'emoicones,
- undo,
- le texte a une valeur initiale (pour permettre le undo) et un v-model pour la valeur courante.

Import: ShowHtml, ChoixEmoji

Dialogue:
- EMmax: vue en plein écran

### McMemo (4)
Attache des mots clés et un mémo à n'importe quel avatar-people, ou groupe dont l'id est connu.

Import: EditeurMd, ApercuMotscles

Dialogue:
- MMedition: gère un ApercuMotscles et un EditeurMd pour afficher / éditer les mots clés et le commentaire à attacher au contact ou groupe.

### ChoixMotscles (1)
Permet de sélectionner une liste de mots clés à attacher à un contact / groupe ou une note.

### ApercuGenx (5)
Présente un aperçu d'un avatar du compte, d'un contact ou d'un groupe.
- ouvre le panel détail d'un contact si ce n'est ni un avatar du compte, ni un groupe. Toutefois si un détail de contact est déjà ouvert, le bouton ne s'affiche pas afin d'éviter des ouvertures multiples.
- ouvre le dialogue ApercuCv sur le bouton zoom.

Import: McMemo

### ApercuAvatar (6)
Affiche les données d'un avatar du compte.
- importé **uniquement* depuis PageCompte.
- édition de la pharse de contact.
- importé **uniquement** depuis PageCompte.

Ce component peut être visible plusieurs fois simultannément (autant qu'un compte a d'avatars).

Import: PhraseContact, ApercuGenx

Dialogue:
- AAeditionpc: édition de la phrase de contact.

### NomAvatar (1)
Saisie d'un nom d'avatar avec contrôle de syntaxe.

### BarrePeople (3)
Affiche trois boutons ouvrant les dialogues / panels associés:
- changement de tranche d'un compte O,
- changement de statuts sponsor d'un compte,
- affiche des compteurs d'abonnements / consommation.

BarrePeople est importé par PanelPeople et PageTranche.

Import: PanelCompta

Dialogues:
- BPchgSp: changement de statut sponsor.
- BPcptdial: affichage des compteurs de compta du compte "courant".
- BPchgTr: changement de tranche.

### ApercuNotif (5)
Affiche une notification. Un bouton ouvre le dialogue DaliogueNotif d'édition d'une notification.

Import: BoutonBulle, ShowHtml, DialogueNotif

Dialogue:
- DNdialoguenotif: dialogue-notif

### PhraseContact (1)
Saisie contrôlée d'une phrase de contact.

### ShowHtml (2)
Ce composant affiche sur quelques lignes un texte en syntaxe MD. 
- un bouton permet de zoomer en plein écran le texte et de revenir à la forme résumé.
- un bouton d'édition est disponible sur option et se limite à émettre un évenement `edit`.

Import: SdDark, SdLight, SdDark1, SdLight1

Dialogue:
- SHfs: vue en plein écran.

### QuotasVols (1)
Affiche l'abonnement en nombre de noye + chat + groupes et de volume de fichier, ainsi que sur option le pourcentage d'utilisation de ces abonnments.
- affiche aussi le quota de consommation (en monétaire) fixé.

### PanelCompta (2)
Affiche les informations d'abonnement et de consommation d'un compte.
- importé par BarrePeople à propos d'un contact,
- importé pat PanelCompta pour les données du compte.

- **Synthèse**: "cumuls" d'abonnement et de consommation correspondant à la période de la création du compte [2023-11-17 17:09] à maintenant (9 jours).
- **Abonnement: nombre de notes + chats + groupes**
- **Abonnement: volume des fichiers attachés aux notes**
- **Contrôle de la consommation**
- **Récapitulatif des coûts sur les 18 derniers mois**
- **Tarifs**

Import: MoisM, PanelDeta

### MoisM (1)
Micro-component de commodité de PanelCompta affichant un bouton affichant 4 mois successifs.

### PanelDeta (1)
Micro-component de commodité de PanelCompta et PanelCredits affichant des compteurs.

### ApercuGroupe (8)
Données d'entête d'un groupe:
- carte de visite, commentaires du compte et mots clés associés par le compte,
- fondateur,
- mode d'invitation,
- hébergement,
- mots clés définis au niveau du groupe.

Import: MotsCles, ChoixQuotas, BoutonConfirm, BoutonHelp, ApercuMembre, ApercuGenx, BoutonMembre, QuotasVols

Dialogues:
- MCmcledit: édition des mots clés du groupe
- AGediterUna: gestion du mode simple / unanime
- AGgererheb: gestion de l'hébergement et des quotas
- AGnvctc: ouverture de la page des contacts pour ajouter un contact au groupe

### ApercuMembre (7)
Afiche une expansion pour un membre d'un groupe:
- repliée: son aperçu de contact et une ligne d'information sur son statut majeur dans le groupe (fondateur, hébergeur, statut).
- dépliée: ses flags et ses date-heures de changement d'état et des boutons pour changer cet état (invitation, configurer, oublier).

Import: InvitationAcceptation, BoutonConfirm, ApercuGenx, BoutonBulle2, BoutonBulle, EditeurMd

Dialogues:
- AMinvit: invitation d'un contact.
- AMconfig: configuration des flags du membre.
- AMoubli: oubli d'un contact jamais invité, ni actif.

### InvitationAcceptation (6)
Formulaire d'acceptation / refus d'une invitation:
- flags d'accès,
- message de remerciement.

Une fiche d'information à propos de l'invitation est obtenue du serveur et contient en particulier les données à propos du ou des invitants.
- on peut accepter une invitation SANS avoir accès aux membres du groupes;
- c'est la fiche invitation qui ramène le row membre correspondant spécifique, sans que la session n'ait eu besoin d'avoir accès aux membres du groupe.

Est invoqué comme dialogue par:
- ApercuMembre: pour les invitations des avatars du compte.
- PageGroupes: pour toutes les invitations en attente inscrites dans les avatars du compte.

Import: EditeurMd, ApercuGenx, BoutonConfirm, BoutonBulle

### ListeAuts (1)
Affiche en ligne la liste des auteurs d'une note avec leur nom ou leur indice memebre et ouvre leur carte de visite sur click.

### NotePlus (6)
Apparaît soit comme un bouton menu, soit comme un bouton proposant l'ajout d'une note selon l'auteur sélectionné.

N'est importé que par PageNotes.

Import: NoteNouvelle

### NoteEcritepar (1)
Bouton dropdown proposant des auteurs possibles:
- pour une note de groupe en création, en édition, pour un fichier,
- pour un item de chat de groupe, l'auteur de l'item

### PanelDialtk (1)
Affiche un ticket de paiement dans laperçu d'un ticket et le panel credits.

### NomGenerique (1)
Saisie d'un nom, de fichier ...

### DialogueNotif (4)
Affichage / saisie d'une notification, texte et niveau.
- enregistrement / suppression selon que la notification est générale, tranche de quoas ou compte.

Import: EditeurMd, BoutonBulle

## Dialogues

### DialogueErreur (1)
Affiche une exception AppExc et gère les options de sortie selon sa nature (déconnexion, continuation ...).

### ChoixEmoji (1)
Dialogue de saisie des émojis à insérer dans un input / textarea.
- se ferme à la fin de la saisie.
- singleton, du fait de son inclusion soit dans EditeurMd soit dans Motscles qui n'ont qu'une seule instance en édition à un instant donné (toujours inclus dans des dialogues).

### PhraseSecrete (1)
Saisie contrôlée d'une phrase secrète et d'une organisation (sur option), avec ou sans vérification par double frappe.

Ce composant héberge *simple-keyboard* qui affiche et gère un clavier virtuel pour la saisie de la phrase. Il utilise pour s'afficher un `<div>` de classe "simple-keyboard" ce qui pose problème en cas d'instantiation en plusieurs exemplaires.
- Ceci a conduit a avoir une seule instance du dialogue hénergée dans App et commandée par la variable sorres.ui.d.PSouvrir
- les propritées d'instantiation sont dans stores.ui.ps, dont ok qui est la fonction de callback à la validation de la saisie.
le dialogue est positionné au *top* afin de laisser la place au clavier virtuel de s'afficher au dessous quand il est sollicité.

PhraseSecrete est ouverte pat :
- PageLogin: saisie de la pharse de connexion.
- AcceptationSponsoring: donnée de la phrase par le filleul juste avant sa connexion.
- PageCompte: changement de phrase secrète.
- PageCompta: saisie de la phrase secrète du Comptable à la création d'un espace.
- OutilsTests: pour tester la saisie d'une phrase secrète et la récupération de ses cryptages.

### ApercuCv (4)
Affiche une carte de visite d'un avatar, contact ou groupe:
- pour un contact, le bouton refresh recharge la carte de visite depuis le serveur.
- pour un avatar ou un groupe, le bouton d'édition permet de l'éditer. Pour un groupe, il faut que compte en soit animateur.

Import: ShowHtml, CarteVisite

### CarteVisite (3)
Dialogue d'édition d'une carte de visite, sa photo et son information.
- est importé **uniquement** depuis ApercuCv (la photo et l'information y étant présente).
- sauvegarde les cartes de visite (avatar et groupe).

Import: EditeurMd

### MotsCles (2)
Edite les mots clés, soit d'un compte, soit d'un groupe.

N'est importé **que** par PageCompte et ApercuGroupe (une seule édition à un instant donné).

Import: ChoixEmoji

### ContactChat (2)
Dialogue de saisie de la phrase de contact d'un avatar, puis création, éventuelle, d'un nouveau chat avec lui.

Import: PhraseContact

### PanelCredits (3)
C'est l'onglet "crédits" de PageCompta:
- affiche les tickets en cours,
- rafraîchit leur incorporation,
- bouton de génération d'un nouveau ticket.

Import: ApercuTicket, PanelDeta, PanelDialtk

### ApercuTicket (2)
Affiche un ticket,
- plié: donnée synthétique,
- déplié: son détail et les actions possibles.

Import: PanelDialtk

### NoteConfirme (2)
Dialogue de confirmation d'une action sur une note:
- ne s'applique qu'à la suppression d'une note.
- vérifie l'autorisation d'écriture, dont l'exclusivité d'accès pour une note de groupe.

Import: BoutonConfirm

### NouveauFichier (2)
Dialogue d'acquisition d'un nouveau fichier, local ou depuis le presse-papier, pour une note.
- permet de changer son nom,
- liste les versions antérieures de même nom devant être purgées.

Import: NomGenerique

## Panels

### DialogueHelp (3)
Affiche les pages d'aide.
- la table des matières, le titre de chaque page, les pages à trouver en bas de chaque page d'aide, sont configurés dans `src/app/help.mjs`
- chaque page d'aide est un fichier par langue dans `src/assets/help`
- les images dans ces pages sont dans `public/help`
- ce dialogue est ouvert / géré par `ui-store pushhelp pophelp fermerhelp`.
- l'ouverture est déclencher par BoutenHelp.

### ApercuChatgr (3)
Affiche le chat d'un groupe.
- ajout d'items et supression d'items.

Import: EditeurMd, NoteEcritepar

### ApercuChat (6)
Affiche le chat d'un avatar du compte avec un contact.
- ajout d'items et supression d'items.
- raccrocher le chat/

Import: SdDark1, EditeurMd, ApercuGenx

### SupprAvatar (2)
Panel de suppression d'un avatar.
- affiche les conséquences en termes de pertes de secrets, de groupes et de chats avec les volumes associés récupérés.
- importé **uniquement* par PageCompte.

### OutilsTests (1)
Trois onglets:
- **Tests d'accés**: tests d'accès au serveur, ping des bases locales et distantes.
- OTrunning:
  - présente la liste des bases synchronisées.
  - sur demande calcul de leur volume (théorique pour le volume V1).
  - propose la suppression de la base.
- **Phrase secrète**: test d'une phrase avec affichage des différents cryptages / encodages associés.

Invoqué par un bouton de la page d'accueil / App.vue

Dialogues:
- OTrunning: affiche la progression du calcul de la taille de la base.
- ORsupprbase: dialogue de confirmation de la suppression.

### NouveauSponsoring (3)
Panel de saisie d'un sponsoring par un compte lui-même sponsor.
- importé par PageSponsorings et PageTranche.

Import: PhraseContact, ChoixQuotas, NomAvatar, EditeurMd, QuotasVols

### AcceptationSponsoring (4)
Saisie de l'acceptation d'un sponsoring, in fine création du compte (si acceptation).
- saisie du nom,
- saisie du mot de remerciement.

Import: EditeurMd, ShowHtml, BoutonHelp, QuotasVols

## PanelPeople (7)
Affiche tout ce qu'on sait à propos d'un contact:
- sa carte de visite et son commentaire / mots clés du compte à son propos.
- la liste des chats auquel il participe, ouvert ou non.
- la liste des groupes (dont le compte a accès aux membres) dont il est membre et son statut.

Si le panel a été ouvert pour ajouter le contact comme membre du groupe courant, un cadre donne le statut de faisabilité de cet ajout et un bouton l'ajoute.

Import: ApercuGenx, ApercuMembre

### PanelMembre (8)
Affiche en panel,
- l'aperçu du contact / avatar membre,
- l'aperçu membre qui donne le détail de son rôle dans le groupe.

N'est ouvert que par un BoutonMembre (depuis ApercuGroupe seulement donc). Est hébergé dans App pour éviter des instantiations multiples.

Import: ApercuGenx, ApercuMembre

### PressePapier (3)
Affiche en panel dans deux onglets,
- les notes gardées en presse-papier,
  - ajout, édition, suppression
- les fichiers gardés en presse-papier
  - ajout, remplacement, suppression, affichage, enregistrement, copie.

Import: ShowHtml, EditeurMd, NomGenerique

### NoteNouvelle (4)
Créé une nouvelle note avec le texte saisi:
- pour un groupe l'auteur de la note est à choisir.

Le rattachement de la note a été défini dans la PageNotes selon l'endroit d'où la nouvelle note a été demandée.

Import: BoutonUndo, EditeurMd, NoteEcritepar

### NoteEdit (6)
Affiche le texte d'une note pour édition:
- pour une note de groupe permet de choisir l'auteur.

Import: EditeurMd, ListeAuts, NoteEcritepar, ApercuGenx

### NoteExclu (6)
Gère l'affichage, l'attribution et le retrait d'exclusivité d'écriture d'une note de groupe à un des membres du groupe.

Import: BoutonBulle, ApercuGenx, ListeAuts

### NoteFichier (3)
Affiche les fichiers attachés à une note:
- gère leur affichage, téléchargement local, suppression.
- gère la gestion en visibilité en mode avion, soirt d'une version spécifique, soit de la version la plus récente.

Import: NouveauFichier, NoteEcritepar

Dialogue:
- NFsupprfichier: confirmation de suppression de fichier
- NFconfirmav1: confirmation visible en mode avion par nom
- NFconfirmav2: confirmation visible en mode avion par version

### NoteMC (6)
Affiche et attribue les mots clés d'une note, personnelle et du groupe.

Import: BoutonBulle, ApercuGenx, ChoixMotscles, ListeAuts

## Pages

### PageLogin (5)
Login pour un compte déjà enregistré ou auto-création d'un compte depuis une phrase de sponsoring déclarée par un sponsor.

Import: PhraseContact, AcceptationSponsoring

### PageAccueil (3)
Affiche:
- un bloc avec tous les accès aux pages s'ouvrant par des icônes de App.
- un second bloc qui est le menu d'accueil accessible depuis la App.

Import: MenuAccueil, BoutonLangue, NotifIcon2, QueueIcon 

### PageCompte (7)
Affiche les avatars du compte et les opérations du compte:
- création d'un nouvel avatar,
- édition des mots clés du compte,
- changement de la phrase secrète.

Import: NomAvatar, ApercuAvatar, MotsCles, SupprAvatar

Dialogues:
- PCnvav: nouvel avatar
- PCchgps: changement de la phrase secrète

### PageChats (7)
Affiche la liste des chats des contacts et des groupes.
- si le filtre filtre.filtre.chats.tous est false, les stores avatar et groupe ne délivrent que ceux de l'avatar courant positionné sur la page d'accueil.
- exporte les chats sélectionnés dans un fichier MarkDown.

Import: ApercuChat, ContactChat, ApercuChatgr, ApercuGenx

### PageClos (3)
Page ouverte sur clôture immédiate de la session:
- blocage intégral par l'administrateur technique,
- compte résilié par une autre session ou celle courante,
- ressort toujours par la déconnexion inconditionnelle.

Import: BoutonBulle, ShowHtml

### PageCompta (7)
Quatre onglets donnant l'état de la comptabilité et des blocages.
- **Notifications**: liste des notifications en cours (avec leurs blocages éventuels).
- **Comptabilité**: abonnement et consommation (PanelCompta).
- **Crédits**: pour les comptes autonomes seulement (PanelCredits).
- **Chats**: chats d'urgence avec le Comptable et les sponsors.

Import: SdAl, ApercuGenx, ApercuNotif, PanelCompta, PanelCredits, ApercuChat

### PageSession (2)
Page qui s'affiche pendant l'initilisation de la session, après login et avant la page d'accueil.
- **Etat général** de la session.
- **RapportSynchro**: son contenu est dynamique lors du chargement de la session, puis fixe après (synthèse du chargement initial).
- **Téléchargements en cours**: zone passive d'affichage sans action. En fin d'intialisation d'une session, les chargements des fichiers accessibles en mode avion et qui ne sont pas disponibles dans la base locale, sont chargés en tâche de fond. Cet zone liste les téléchargements restant à effectuer.
- **Téléchargements en échec**: erreurs survenues dans ces téléchargements. Actions possibles sur chaque fichier en échec: _ré-essai abandon_.

Import: RapportSynchro

### PageEspace (4)
Affiche le découpage de l'espace en tranches:
- pour le Comptable, création de tribu et ajustement des paramètres de l'espace pour les transferts de compte O / A.

La page est également invoquée dans un dialogue interne de PageAdmin pour affichage des tranches (mais sans droit d'agir).

Import: ChoixQuotas, TuileCnv, TuileNotif, ApercuNotif

## PageAdmin (5)
C'est LA page de l'administrateur technique.
- 2 boutons techniques: lancer un GC, afficher le dernier rapport de GC.
- un boutons fonctionnel: créer une organisation.
- un bouton de rafraîchissement.

Liste les organisations existantes:
- affichage du détail de leurs tranches sur bouton.
- changement de profil.
- création / gestion de la notification sur l'espace.

Import: ApercuNotif, PageEspace

### PageTranche (6)
Affiche en tête la tranche courante,
- celle du compte
- pour le comptable celle courante sélectionnée depuis la PageEspace.
- pour le comptable ouvre le panel NouveauSponsoring pour sponsoriser un compte dans n'importe quelle tranche.

Affiche en dessous les sponsors et pour le Comptable les autres comptes de la tranche.

Import: TuileCnv,TuileNotif, ApercuNotif, ChoixQuotas, ApercuGenx, PanelCompta, QuotasVols, NouveauSponsoring, BarrePeople

Dialogues: 
- PTcptdial : affichage des compteurs comptables du compte sélectionné
- PTedq: mise à jour des quotas du compte sélectionné

### PageSponsorings (4)
Liste les sponsorings actuellement en cours ou récents:
- boutons de prolongation des sponsorings en cours et d'annulation.

Bouton général pour créer un nouveau sponsoring.

Import: NouveauSponsoring, ShowHtml, QuotasVols

### PageGroupes (8)
Liste les groupes accédés par le compte, dans lesquels il est actif.
- synthèse des volumes occupés par les groupes hébergés,
- bouton de création d'un nouveau groupe,
- une carte par groupe avec :
  - un bouton pour ouvrir le chat du groupe,
  - un bouton pour accéder à la page du groupe.

Import: ChoixQuotas, NomAvatar, ApercuGenx, InvitationsEncours, ApercuChatgr

Dialogue:
- PGcrgr: création d'un groupe.

### PageGroupe (10)
Affiche les détails d'un groupe:
- onglet **Détail du groupe**: entête et participations des avatars du compte au groupe.
  - bouton d'ajout d'un contact comme contact du groupe.
- onglet **Membres**: liste des contacts membres du groupe si le compte a accès aux membres.

Import: ApercuMembre, ApercuGroupe

### PagePeople (6)
Affiche tous les contacts connus avec une courte fiche pour chacun (pouvant ouvrir sur le détail du contact).
- un bouton rafraîchit les cartes de viste qui en ont besoin.

Import: ApercuGenx

### PageNotes (7)
Affiche l'arbre des notes avec pour racines les avatars et les groupes:
- en tête affiche le détail de la note courante, avec les actions qu'elle peut subir.
- la barre séparatrice petmet de lancer le chragment local des notes sléctionnées et le plier / déplier global de l'arbre.
- en bas l'arbre des notes selon leur rattachemnt.

Import: ShowHtml, ApercuMotscles, NoteEdit, NoteMc, NotePlus, NoteExclu, NoteFichier, NoteConfirme, ListeAuts

Diaalogue:
- PNdl: dialogue gérant le chargement des notes en local.

### PageFicavion (2)
Affiche la liste des fichiers visible en mode avion et pour chacun,
- permet de l'afficher et de l'enregistrer localement,
- de voir la note à laquelle il est attaché.

# Données en mémoire _modèle_ et persistantes localement IDB

# Opétrations, connexions, synchronisation et autres

# En chantier

### A développer / revisiter
- pour la doc (Documents.md) vérifier et écrire les conditions de **Prise d'hébergement**.
- blocage des accroissements de volume: vérifier le blocage
- GC à réviser: Tickets Chatgrs à prendre en compte

### Features à développer
_Arrêtés mensuels des Tickets_ (**CSV**)
- tickets réceptionnés dans le mois pour le Comptable (gestion des archives mensuelles).
  - une ligne par ticket dont l'ids débute par le mois.
  - une fois archivé dans un secret du Comptable, appel d'opération du serveur pour détruire tous les tickets du mois et antérieurs.

_Photos périodiques / sur demande_ des abonnements / consommation des comptes
- une ligne par comptas d'un extrait des compteurs relatifs au mois M-1 (dès qu'il est figé).

# Boîtes à secrets - Client

## Session d'un compte
Une session est associée à un compte connecté (ou en connexion). Elle dispose de données :
- en mémoire store/db, toujours,
- sur IDB, seulement si la session est synchronisé ou avion.

Les tables suivantes se retrouvent :
- trois singletons dont la clé est l'id du compte : `compte prefs compta`. L'id en IDB est par convention 1.
- trois collections d'objets _maîtres_:
  - les objets `avatar` cités dans le compte : la propriété `mack` donne leur nom / clé (donc id).
  - les objets `couple` cités dans les avatars cités ci-dessus: la propriété lcpk donne leur clé (donc id).
  - les objets `groupe`  cités dans les avatars cités ci-dessus: la propriété lgrk donne leur nom / clé (donc id).
- des collections d'objets secondaires : leur propriété id est l'id d'un objet maître et ils ont une seconde partie de clé pour les distinguer :
  - objets `membre` secondaires de `groupe`, clé secondaire `im` (indice membre).
  - objets `secret` secondaires de `avatar, couple, groupe`, clé secondaire `ns` (numéro de secret).

Les objets `invitgr` secondaires de `avatar` sont transients en session : récupérés en synchronisation ils déclenchent une opération de mise à jour de leur avatar pour qu'il référence le groupe dans `lgrk`. Ces objets ne sont ni mémorisés, ni en store/db, ni en IDB.

Les objets contact sont demandés explicitement par une vue et ne sont ni mémorisés, ni en store/db, ni en IDB.

#### Les cartes de visite
Tout objet maître `avatar / groupe / couple` a un objet `cv` de même id :
- ils sont créés simultanément.
- la propriété `x` de `cv` quand elle est > 0, indique que l'objet maître est _disparu_ et a été purgé.

### Structure en mémoire : store/db
Son `state` (vuex) détient:
- les singletons : `compte prefs compta`
- les collections d'objets maîtres : une map de clé id pour donner l'objet correspondant
  - `avatars[1234]` : donne l'avatar d'id `1234`
- les collections d'objets secondaires secret membre
  - `secrets@1234[56]` donne le secret d'id `1234, 56`
  - `secrets@1234` donne la map de tous les secrets du même maître `1234`
- un singleton particulier `repertoire` (voir plus loin) qui est une map de toutes les cvs des `avatar / groupe / couple`.
- un singleton particulier `sessionsync` (voir plus loin)).

Son state détient aussi des _objets courants_ dans les vues :
- `avatar`: l'avatar courant
- `groupe`: le groupe courant
- `groupeplus`: le couple courant [groupe, membre] ou membre est celui de l'avatar courant
- `couple`: le couple courant
- `secret`: le secret courant

Si par exemple l'objet du groupe d'id `123` change,
- `groupes[123]` change et référence la nouvelle valeur de l'objet,
- `groupe` change et référence la nouvelle valeur de l'objet.

#### Répertoire des CVs
La classe `Repertoire` est une simple map avec entrée par carte de visite (de clé id de la carte de site).

Chaque entrée a des propriétés additionnelles par rapport à cv :
- **pour une cv d'avatar:**
  - `na`: son `NomAvatar` (couple non / clé),
  - `lgr`: la liste des ids des groupes dont l'avatar est membre,
  - `lcp`: la liste des ids des couples dont l'avatar est, soit le conjoint _interne_ (avatar du compte), soit le conjoint _externe_ (l'avatar n'est pas un avatar du compte).
  - `avc`: `true` si c'est un avatar du compte
- **pour une cv de groupe:**
  - `na`: son `NomAvatar` (couple non / clé)
  - `lmb`: la liste des ids des avatars qui en sont membre.
- **pour une cv de couple:**
  - `na`: son `NomAvatar` (couple non / clé). Son nom est construit depuis ceux de son ou ses conjoints.
  - `idE`: l'id du conjoint externe (s'il est connu),
  - `idI`: l'id du conjoint interne.

**La session détient un objet `rep` qui est l'image, non réactive** (de travail) du répertoire.

**store/db détient la propriété `repertoire`, image réactive du répertoire, stable**, celle qui a été fixée par l'opération commit() du répertoire rep de la session.

### La base locale IDB
Il y a une base par compte.

Elle contient les tables `compte prefs compta avatar groupe couple secret cv` :
- la clé primaire de chacun est,
  - pour les singletons `compte prefs compta` : `1`
  - pour les objets maîtres `avatar groupe couple` le cryptage par la clé K du compte de leur id en base64.
  - pour les objets secondaires le couple `id id2`:
    - `id` : le cryptage par la clé K du compte de l'id en base64 de son maître.
    - `id2` : le cryptage par la clé K du compte de son id relative (`ns` ou `im`) à son maître.
- la propriété `data` est le cryptage par la clé K du compte de la sérialisation de l'objet.

Les tables ont donc deux 2 propriétés `id data` ou 3 propriétés `id, id2,  data`.

**IDB est toujours cohérente** : les opérations spéciales de mise à jour accumulent dans leur traitement les mises à jour pour IDB dans leur objet `OpBuf` et un seul commitRows() intervient à la fin pour enregistrer toutes les mises à jour en attente dans `OpBuf`.

#### La table `sessionsync`
Cette table enregistre les date-heures,
- de la session synchronisée précédente correctement connectée puis terminée : `dhdebut dhfin`
- de la session synchronisée en cours : 
  - `dhlogin` : dh de fin de login,
  - `dhsync` : date-heure de fin de la dernière opération de synchronisation,
  - `dhpong` : date-heure de réception du dernier _pong_ reçu sur le websocket attestant que celui-ci n'est pas déconnecté.

Cet objet est disponible dans store/db `sessionsync`, uniquement quand la session courante est _synchronisée_.

## Opérations et cohérence des états store/db et IDB
#### Trois opérations d'initialisation
Il y a 3 opérations d'initialisation des données de session :
- **`ConnexionCompte`**
  - elle reconstitue en mémoire l'état des données du compte depuis,
    - IDB si c'est une session _synchronisée_ ou _avion_,
    - et le serveur si c'est une session _synchronisée_ ou _incognito_,
    - l'état est cohérent et les objets inutiles ont été purgés,
    - l'état des données est transcrit sur IDB par une unique transaction (commitRows()) et sur store/db par un unique appel de fonction.
- **`CreationCompte`** (sans parrain) et **`AcceptationParrainage`** (avec parrain).
  - elles sont plus simples puisqu'il n'y a par définition aucun état antérieur à prendre en compte et très peu d'objets à commiter dans IDB et store/db.

La propriété state/ui `statutsession` vaut
- 0 : avant exécution de ces initialisations de sessions,
- 1 : pendant l'exécution de celles-ci,
- 2 : après la fin de l'exécution (succès, sinon retour à 0 et pas de session)
- 0 : en cas de déconnexion accidentelle ou explicite (plus de session).

#### Unique opération de mise à jour : ProcessQueue
Il n'y a qu'une seule opération de mise à jour (après initialisation), ProcessQueue :
- elle ne s'exécute que quand l'initialisation complète de la session est faite (statutseesion à 2).
- les messages de mise à jour reçus sur WebSocket sont stockées en queue dans l'ordre d'arrivée.
- la queue est traitée (si elle n'est pas vide),
  - dès que le statut de session passe à 2
  - dès qu'un message de synchronisation est reçu et que le statut de session est 2.
Comme pour les opérations d'initialisation, les modifications sont stockées dans OpBuf et validées en un seul appel à la fin.

#### Autres opérations
Aucune autre opération ne fait de mises à jour des données store/db et IDB, ni ne lit IDB.

> _Remarque_ : les objets courants de navigation store/db avatar groupe ... peuvent changer au gré des navigations mais sans mise à jour de leur contenu.

En conséquence les actions UI peuvent accéder à tout instant à un état cohérent des données en store/db qui n'évolue que par le commit de ProcessQueue (d'état cohérent en état cohérent).

#### Début et fin d'une session, inter-session
**Une session existe dès qu'une opération de connexion ou de création de compte débute :**
- l'état de session est disponible dans la variable `data` de `modele.mjs`et subsiste dure jusqu'à la **déconnexion**, explicite ou accidentelle.
- une session est caractérisée par un compte.
- dès que l'initialisation (connexion / création de compte) s'est bien terminée l'état de ses données est cohérent.
- le **traitement des notifications des mises à jour** du serveur par `ProcessQueue` maintient l'état store/db et IDB à jour et cohérent.

Une session a quelques propriétés traduisant son état interne, en plus des données _métier_ en store/db et IDB.

Durant une session les pages peuvent être : `synchro` (durant l'initialisation), puis `compte` ou `avatar`.

#### Inter-session
Entre 2 sessions, très peu d'information est disponible :
- `org` : l'organisation choisie (quand elle l'a été).
- `mode` : `synchronisé incognito avion` (quand il a été choisi).

En inters-session les pages ne peuvent être que `org` ou `login`.

#### Propriétés d'une session
### Objet OpBuf
Cet objet créé en début des opérations ci-dessus stocke les mises à jour en attente collectées pendant l'opération afin de pouvoir réaliser à la fin :
- une mise à jour de store/db en un seul appel,
- une écriture sur IDB en une seule transaction.

Ceci permet d'éviter de faire apparaître,
- en store/db des états intermédiaires incohérents propres à perturber l'affichage,
- en IDB un état fonctionnellement incohérent résultant d'une mise à jour partielle.

### Opération `ConnexionCompte`
Elle a pour objectifs :
- de s'assurer que le compte correspondant à la phrase secrète saisie existe,
- d'alimenter en store/db et en IDB toutes les données du périmètre du compte.

C'est la seule opération qui lit IDB afin de récupérer un maximum de données sans avoir à les obtenir du serveur.

Elle écrit sur IDB les données récupérées du serveur de manière à ce que son état soit cohérent et propre à être utilisé par une connexion ultérieure en mode _avion_.

Elle purge d'IDB les données _inutiles_ (obsolètes ou sorties du périmètre du compte).

- **suppression éventuelle de IDB** si c'est l'option demandée au login.
- si c'est une connexion en mode _avion_, vérifie qu'une propriété du `localstorage` donne le nom de la base du compte pour la phrase secrète saisie.
- **connexion effective :**
  - attribue à la session un `sessionId` aléatoire
  - ouverture de IDB (sauf en mode _incognito_)
  - création d'un websocket avec le serveur (sauf en mode _avion_), lancement du ping pong.
- **phase itérative 0,1,2**
  - tant que ces trois phases ne se sont pas déroulées sans incident, on boucle (5 fois avant exception).
  - **phase 0 : récupération de `compte / prefs / compta`** : ceci donne une liste d'avatars (dans `lgrk` de compte). Signature (dans compta) et abonnement au compte.
  - **phase 1 : pour tous les avatars cités dans le compte**
    - récupération de `avatar`
    - signature (dans cv) et abonnement
    - _si la version du compte a changé_, donc qu'il a pu avoir un avatar en plus ou en moins depuis la phase 0, échec et on reprend la phase 0-1-2
    - ceci donne la liste des groupes et couples du périmètre du compte.
  - **phase 2 : pour tous les groupes et comptes du périmètre,**
    - récupération du `couple groupe`
    - signature (dans cv) et abonnement
    - si l'une des versions, du compte ou d'un des avatars récupérés a changé (possibilité de groupe / couple en plus ou en moins), échec et on reprend la phase 0-1-2
- **phase 3. Récupération des membres et secrets** rattachés aux `avatar / couple / groupe` récupérés ci-dessus. Comme on est abonné à ces objets, s'ils changent les modifications seront traitées en synchronisation.
- **phase 4. Récupération des cv des objets maîtres et des membres rattachés.**
  - l'objet `sessionsync` a une propriété `vcv` qui donne la plus haute version des cv récupérées et stockées en IDB.
  - liste `vp` : ce sont les cv référencées dont on a déjà une copie (soit parce que son x indique une disparition, soit parce qu'une cv a été déclarée). On récupère du serveur celles ayant une version supérieure à `vcv`.
  - liste `vz` : ce sont les cv référencées dont on n'a jamais eu copie antérieurement. On récupère du serveur celles ayant une version non 0. Une cv avec une version 0 indique seulement que l'objet existe et qu'il n'a jamais eu de cv.
  - la plus haute version des cv récupérées donne le `vcv` pour la prochaine session.
- **phase 5. Récupération des fichiers attachés déclarés stockés localement**, manquants ou ayant changé de version.
- **phase 6. Récupération des invitations aux groupes (invitgr) concernant un des avatars du compte**. Ces objets ne sont pas stockés et donne lieu immédiatement à une requête pour modifier les avatars correspondants et leur faire référencer les avatars du compte.
- **finalisation :**
  - les mises à jour store/db sont validées,
  - les mises à jour dans IDB sont soumises et commitées en une seule opération.
  - la fin de la connexion est actée : la session passe en statutsession 2 (apte à fonctionner).

### Opération de synchronisation `ProcessQueue`
Chaque opération traite un ou plusieurs lots de notifications envoyées par le serveur sur websocket.
- tous les objets modifiés sont collectés et seule la plus haute version pour chaque id est conservée pour traitement.
- l'objectif est de mettre à jour :
  - store/db en une seule fois à la fin.
  - IDB en une seule transaction à la fin.

## Pages
### Org : `/`
Page racine permettant de choisir son organisation.  
On revient à cette page si dans l'URL on n'indique pas de code organisation ou un code non reconnu.

### Login : `/_org_`
Page de connexion à un compte ou de création d'un nouveau compte.

### Synchro : `/_org_/synchro`
Dès l'identification d'un compte, cette page s'affiche. Elle ne permet pas d'action mais affiche l'état de chargement / synchronisation du compte.  
Elle enchaîne, 
- a) soit sur la page `Compte` an cas de succès, 
- b) soit en retour vars la page `Login` en cas de déconnexion.

### Compte : `/_org_/compte`
Dès que les données du compte sont complètement chargées, cette page s'affiche et donne la synthèse du compte, la liste de ses avatars...

Navigations possible :
- `Login` : en cas de déconnexion
- `Avatar` : vers l'avatar _sélectionné_

### Avatar : `/_org_/avatar`
Détail d'un avatar du compte.

Navigations possibles :
- `Login` : en cas de déconnexion
- `Compte` : retour à la synthèse du compte.

### Panneau latéral Menu
Infos et boutons d'actions (affichage de boîtes de dialogue, etc.)

## Actions et opérations
### Actions
Elles n'affectent que l'affichage, la visualisation des données. Elles ne changent pas l'état des données du compte, ni sur IDB ni sur le serveur central.

Elles ne font pas d'accès ni à IDB ni au réseau, sauf l'action spéciale de **ping** :
- _ping du serveur_ : pas en mode _avion_.
- _ping DB de la base de l'organisation sur le serveur_ : il faut que cette organisation soit connue, pas en mode _avion_.
- _sélectionné_ : il faut que le compte soit identifié, pas en mode _incognito_.

### Opérations
Seules les 4 opérations spéciales vues antérieurement modifient l'état des données du compte, en mémoire et **sur IDB et/ou le serveur**.

**Les opérations UI _standard_** ne changent pas létat store/db ni IDB : elles postent des requêtes au serveur.

Une opération s'exécute toujours dans le cadre d'une **session**, c'est à dire avec un **compte identifié ou en cours d'identification** (donc pas forcément encore authentifié ni créé).
- si la session est en mode synchronisé ou incognito, une session WebSocket est ouverte.
- si la session est en mode synchronisé ou avion, la base IDB est accessible et ouverte.

#### Opérations `UI` _standard_ initiées par UI
C'est le cas de l'immense majorité des opérations : elles sont interruptibles par l'utilisateur (quand il en a le temps) par appui sur un bouton.

Aucune action ou nouveau lancement d'opérations ne peut avoir lieu quand une opération UI est déjà en cours, sauf la demande d'interruption de celle-ci.

Trois événements peuvent interrompre une opération UI :
- l'avis d'une rupture d'accès au réseau (WebSocket ou sur un accès POST / GET).
- l'avis d'une impossibilité d'accès à IDB.
- une demande d'interruption de l'utilisateur.

La détection d'un de ces événements provoque une exception BREAK qui n'est traitée que sur le catch final de l'opération (en cas de _catch_ elle doit être re-propagée telle quelle).

Le traitement final du BREAK consiste à dégrader le mode de la session en **Avion** ou **Visio** ou **Incognito** selon les états IDB et NET (s'il ne l'a pas déjà été).

#### Opération `ProcessQueue` `WS` _WebSocket_ initiées par l'arrivée d'un message sur WebSocket
Une seule opération de ce type peut se dérouler à un instant donné.

Elles ne sont pas interruptibles, sauf de facto par la rupture de la liaison WebSocket (voire en conséquence d'une action de déconnexion).

Deux événements peuvent interrompre une opération WS :
- l'avis d'une rupture d'accès au réseau (WebSocket ou sur un accès POST / GET).
- l'avis d'une impossibilité d'accès à IDB.

La détection d'un de ces événements provoque une exception BREAK qui n'est traitée que sur le catch final de l'opération (en cas de _catch_ elle doit être re-propagée telle quelle sans traitement). 

Le traitement final du BREAK consiste à dégrader le mode de la session en **Avion** ou **Visio** selon les états IDB et NET (s'il ne l'a pas déjà été).

> In fine `ProcessQueue` ne sort jamais en exception.

### Boîtes de dialogue
##### Publiques
Elles peuvent être commandées d'ailleurs de la vue qui la contient : principalement celles du _layout_ mais aussi quelques autres.

**Leur affichage est toujours commandée par une variable de store/ui** : dans n'importe quel endroit du code il suffit donc de basculer leur variable d'affichage pour que la boîte apparaisse.

Sur la mutation `majstatutsession` avec un statut non 2 (ok), toutes les boîtes publiques sont fermées.

##### Spécifiques d'une vue
Chaque boîte dépend d'une variable définie au setup() par def() :
- la variable `sessionok` (ou plus `statutsession`) active est déclarée.
- `watch` permet de fermer toutes les boîtes.

    setup () {
    const mcledit = ref(false)
    const sessionok = computed(() => { return $store.state.ui.sessionok })
    watch(() => sessionok.value, (ap, av) => {
      if (ap) {
        mcledit.value = false
      }})

Ainsi :
- dans l'affichage des `v-if="sessionok"` permettent de ne rien afficher lors d'une fermeture de la vue quand les objets qu'elle affiche ont déjà disparu.
- les boîtes de dialogues se ferment automatiquement en cas de déconnexion.

## Modes
### Avion
- pas d'accès au réseau, pas de session WebSocket.
- les seules opérations possibles mettent à jour IDB : enregistrement de secrets pour mise à jour différée à la prochaine synchronisation.
- en cas de perte d'accès à IDB, le mode est dégradé en **Visio**.

### Incognito
- pas d'accès à IDB, session WebSocket ouverte.
- les seules opérations impossibles sont celles devant lire / écrire IDB (enregistrement de textes en attente, de fichiers à attacher plus tard et de fichiers attachés stockés localement).
- en cas de perte d'accès au réseau (session WS fermée), le mode est dégradé en **Visio**. 

### Synchronisé
- accès à IDB, session WebSocket ouverte.
- toutes opérations possibles.
- en cas de perte,
  - d'accès au réseau, le mode est dégradé en mode **Avion**.
  - d'accès à IDB, le mode est dégradé en mode **Incognito**.
  - des deux, le mode est dégradé en mode **Visio**.

### Visio : mode dégradé
- aucun accès, ni à IDB, ni au réseau, pas de session WebSocket.
- aucune opération possible
- on ne choisit jamais le mode Visio : il résulte d'une dégradation des trois autres modes.

En cas de tentative de reconnexion d'un compte, celle-ci s'effectue dans le mode initial choisi par l'utilisateur, pas dans le mode _dégradé_.

La session d'un compte comporte donc deux modes :
- le mode _initial_,
- le mode _courant_, qui s'il diffère du mode initial, résulte d'une dégradation.

### Dégradation d'un mode
Elle s'effectue automatiquement. Toutefois l'utilisateur reçoit un avis lui demandant s'il préfère,
- une déconnexion franche, 
- tenter une re-connexion,
- ou rester dans le mode dégradé.

## Session
Il y a ouverture de session dès qu'il y a une intention d'identification / création d'un compte : c'est une opération qui créé la session (en tout début).
- la session a une sessionId qui l'identifie (aléatoire sur 6 octets).
- la session a un compte : mais au début de l'opération créatrice il peut être vide.
- la session a deux statuts :
  - IDB : ok ou pas.
  - NET : ok ou pas.
- le mode courant de la session est invariant dans sa vie, son mode courant peut évoluer. En mode Visio aucune opération n'est possible.

Une session peut avoir au plus deux opérations en cours : une UI et une WS

Une session est détruite par une action de `deconnexion`.

L'action de `reconnexion` sur une session source recrée une autre session :
- la session source doit être en statut KO en IDB, NET ou les deux.
- la nouvelle session a le mode initial de la session source qui est détruite.
- la nouvelle session a pour compte le compte de la session initiale (donc authentifié).

## `localStorage` et IDB
**En mode *avion*** dans le `localStorage` les clés `monorg-hhh` donne chacune le numéro de compte `ccc` associé à la phrase de connexion dont le hash est `hhh` : `monorg-ccc` est le nom de la base IDB qui contient les données de la session de ce compte pour l'organisation `monorg` dans ce browser.

**En mode *synchronisé***, il se peut que la phrase secrète actuelle enregistrée dans le serveur (dont le hash est `hhh`) ait changé depuis la dernière session synchronisée exécutée pour ce compte :
- si la clé `monorg-hhh` n'existe pas : elle est créée avec pour valeur `monorg-ccc` (le nom de la base pour le compte `ccc`).
- si la base `monorg-ccc` n'existe pas elle est créée.
- l'ancienne clé, désormais obsolète, pointe bien vers le même compte mais ne permet plus d'accéder à ce compte, dont la clé K a été ré-encryptée par la nouvelle phrase.

## Barre de titre
A droite :
- icône menu : ouvre le menu
- icône home : provoque le retour à Accueil mais demande confirmation de la déconnexion.
- icône et nom de l'organisation

A gauche :
- icône donnant le mode _synchronisé incognito avion_ : un clic explique ce que signifient les modes.
- icône donnant la phase :
  - pas connecté (carré blanc)
  - en synchro (flèches tournantes)
  - en travail :
    - rond vert : mode actif
    - verrou rouge : mode passif (mise à jour interdite).
  - un clic explique ce que signifie l'état

# Gestion des fichiers disponibles en mode avion
- Chaque fichier est identifié par `idf` (grand nombre aléatoire).
- Chaque fichier est attaché à un **secret** identifié par `id ns`, ne peut en changer et est immuable.
- Un fichier peut être détruit, pas mis à jour.

### Cache des fichiers en IDB
Deux tables gèrent ce stockage en IDB :
- `fdata` : colonnes `idf, data`. Seulement insertion et suppression
  - `data` est le contenu du fichier tel que stocké sur le serveur (crypté / gzippé).
- `fetat` : colonnes `idf, data : {dhd, dhc, lg, nom, info}`. Insertion, suppression et mise à jour.
  - `dhd` : date-heure de demande de chargement.
  - `dhc` : date-heure de chargement.
  - `lg` : taille du fichier (source, son v2).
  - `nom info` : à titre d'information.

#### Transactions
- `fetat` peut subir une insertion sans mise à jour de `fdada`.
- `fetat` peut subir une suppression sans mise à jour de `fdata` si le row indique qu'il était encore en attente (`dhc` 0).
- `fetat` et `fdata` peuvent subir une suppression synchronisée.
- quand `fdata` subit une insertion, `fetat` subit dans la même transaction la mise à jour de `dhc`.

La table `fetat` est,
- lue à l'ouverture d'une session en modes _synchronisé_ et _avion_ (lecture seule),
- l'état _commité_ est disponible en mémoire durant toute la session.

#### Démon
En mode synchronisé un démon tourne en tâche de fond pour obtenir du serveur les fichiers requis pas encore disponibles en IDB.

Dans la barre de statut en haut, l'icône du mode synchronisé est en _warning_ quand il y a des fichiers en attente / chargement. Quand on clique sur cette icône, la liste `fetat` est lisible (avec l'indication en tête du fichier éventuellement _en cours de chargement_).

#### État de disponibilité pour le secret _courant_
Dans db/store, le state `dispofichiers` est synchronisé avec db/secret (le secret _courant_). Le démon maintient dans `dispofichiers` la liste des idf _en cours de chargement_ (en fait demandés mais pas encore chargés). La page des fichiers attachés du secret courant peut ainsi afficher si le fichier est disponible localement ou non :
- en mode _avion_ ceci indique si il sera ou non affichable,
- en mode _synchronisé_ ceci indique si son affichage est _gratuit_ (et immédiat).

## Objets Secret et AvSsecret
Les objets `Secret` sont ceux de classe `Secret` disponibles dans le store/db.
- Identifiant : `[id, ns]`
- Propriété `mfa` : c'est une map,
  - _clé_ : `idf`, l'identifiant d'un fichier attaché,
  - _valeur_ : `{ nom, lg, dh ... }`, nom externe et taille. Pour un nom donné pour un secret donné il y a donc plusieurs versions de fichier chacune identifiée par son idf et ayant une date-heure d'insertion dh. Pour un nom donné il y a donc un fichier _le plus récent_.

Un objet de classe `AvSecret` existe pour chaque secret pour lequel le compte a souhaité avoir au moins un des fichiers attachés disponible en mode avion.
- Identifiant : `[id, ns]`
- Propriétés :
  - `lidf` : liste des identifiants des fichiers explicitement cités par leur identifiant comme étant souhaité _hors ligne_.
  - `mnom` : une map ayant,
    - _clé_ : `nom` d'un fichier dont le compte a souhaité disposer de la _version la plus récente_ hors ligne.
    - _valeur_ : `idf`, identifiant de cette version constaté dans l'état le plus récent du secret.

Chaque objet de classe `AvSecret` est disponible dans db/store avec la même structure que pour secret :
- une entrée `avsecret@id` qui donne une map de clé `ns` pour chaque objet `AvSecret`.

`AvSecret` est maintenu en IDB à chaque changement :
- la clé primaire `id,id2` est comme celle de `Secret` cryptée par la clé K du compte.
- _data_ est la sérialisation de `{lidf, mnom}` cryptée par la clé K du compte.

En mode _synchronisé_ et _avion_ tous les `AvSecret` sont chargés en mémoire dans db/store.

### Mises à jour des AvSecret
Il y a deux sources de mise à jour :
-**(a) le compte fait évoluer ses souhaits**, modifie `lidf / mnom` : il peut en résulter,
  - une liste d'idf à ne plus conserver en IDB,
  - une seconde liste d'idf qui ne l'étaient pas et doivent désormais l'être. 
  - Ces deux listes sont calculées par comparaison entre la version _actuelle_ d'un `AvSecret` et sa version _future_ (désormais souhaitée).
-**(b) une mise à jour d'un `Secret`** peut faire apparaître des incohérences avec l'`AvSecret` correspondant (quand il y en a un) et qui doit se mettre en conformité:
  - des idf cités dans `lidf` n'existent plus : ils doivent être supprimés.
  - pour un nom dans `mnom`, l'idf cité n'est plus le plus récent (il doit être supprimé **et** un autre devient requis et _peut-être_ non stocké donc inscrit comme _à charger_).
  - des noms cités dans `mnom` n'existent plus, ce qui entraîne la disparition des idf correspondants (et de l'entrée `mnom`).
  - en conséquence il résulte de la comparaison entre un `Secret` et son `AvSecret` correspondant :
    - une liste d'idf à supprimer dans `fetat fdata` ce qui est fait sur l'instant.
    - une liste d'idf _à charger_ et noté dans `fetat` sur l'instant, le chargement effectif étant effectué à retardement par le démon.
    - une mise à jour de l'`AvSecret` pour tenir compte des contraintes du nouveau `Secret`, voire sa suppression si Secret est supprimé **ou** que `lidf` et `mnom` sont vides.

**Les traitements (a) sont effectués quand le compte en a exprimé le souhait par action UI**. Ils sont immédiats pour les suppressions mais les chargements de nouveaux idf seront traitées par le démon avec retard.

**Les traitements (b) sont effectués** :
- en fin d'initialisation de la session en mode _synchronisé_ quand tous les `Secret` sont chargés : il détecte,
  - les mises à jour éventuelles de chaque `AvSecret` pour chaque objet `Secret` existant correspondant.
  - les suppressions des `AvSecret` (et donc des idf dans `fdata / fetat`) pour chaque `AvSecret` pour lequel il n'existe plus de `Secret` (existant) associé.
- **à la synchronisation pour chaque mise à jour / suppression** d'un `Secret` entraînant le cas échéant une mise à jour ou une suppression de l'`AvSecret` correspondant.

# Données en mémoire, réactives (stores) ou non
#### `config-store`
Données de configuration récupérées au boot (par `boot/appconfig.js`) de `assets/config/app-config.json`

#### `session-store`
État courant de la session active (cf ci-dessus).

#### `avatar-store`
Map par id d'un avatar des avatars du compte (objet de classe Avatar)

## Modèle des données en mémoire
Il est consitué :
- des stores, pour toutes les données susceptibles d'être affichées ou surveillées.
- du répertoire des cartes de visite.

#### Répertoire des cartes de visite
##### Classe : `NomGenerique`
- 4 variantes : `NomAvatar`, `NomGroupe`, `NomContact`, `NomTribu`
- propriétés :
  - `nom` : nom court immuable de l'objet
  - `rnd` : u8(32) - clé d'encryption
  - `id` : 
    - pour tous les objets sauf le Comptable : `hash(rnd)`. Hash entier _js safe_ de rnd.
      - type: reste de la division par 4 de l'id: 0:avatar 1:contact 2:groupe 3:tribu
    - pour le Comptable : `IDCOMPTABLE` de api.mjs : `9007199254740988`

##### Map statique `repertoire` de `modele.mjs`
Fonctions d'accès
- `resetRepertoire ()` : réinitialisation
- `getCle (id)` : retourne le rnd du nom générique enregistré avec cette id
- `getNg (id)` : retourne le nom générique enregistré avec cette id

`repertoire` a une entrée pour :
- 1-chaque avatar du compte,
- 2-chaque groupe dont un avatar du compte est membre
- 3-chaque contact dont un avatar du compte est conjoint interne
- 4-chaque avatar externe,
  - membre d'un des groupes ci-dessus,
  - conjoint externe d'un des contacts ci-dessus

Chaque entrée a pour clé `id` et pour valeur `{ ng, x }`
- `ng` : objet `NomGenerique`
- `x` : statut de disparition, `true` si disparu sinon `undefined` (vivant).

Une entrée de répertoire est quasi immuable : la seule valeur qui _peut_ changer est `x` : inscrite à la connexion du compte, elle n'est mise à jour par synchronisation (c'est le GC qui le positionne au plus une fois).

Le répertoire grossit en cours de session mais ne se réduit jamais. Il contient des objets "obsolètes":
- avatar du compte détruit.
- groupe n'ayant plus d'avatars du compte membre.
- contact quitté par leur conjoint interne.
- avatar externe n'étant plus membre d'aucun groupe ni conjoint externe d'aucun couple.

# Annexe: l'arbre des notes
Remarques et règles de gestion applicables à l'arbre des notes `note-store.js`

Une note peut être :
- top : elle est rattachée directement à son avatar ou groupe.
- rattachée à une autre note définie par :
  - rid : l'id de la note de rattachement,
  - rids : l'ids de la note de rattachement,
  - rnom : quand rid est une id de groupe, le nom de son groupe.

**Règles :**
- une note de groupe ne peut être rattachée qu'à une note du même groupe,
- une note d'avatar peut être rattachée,
  - soit à une note du même avatar,
  - soit à une note de groupe.
- donc un arbre de groupe peut contenir des notes de lui-même et d'avatars: les sous-arbres dont la tête est une note d'avatar n'ont que des notes du même avatar.

### Racines _avatar_
- des notes d'avatar ne peuvent parvenir qu'après que leur avatar ait été connu.
- les racines _avatar_ sont toujours réelles.

### Racines _groupe_, réelles et zombi
- des notes de groupes ne peuvent parvenir qu'après que son groupe ait été déclaré : leur racines _groupe_ sont réelles.
- mais des notes d'avatars peuvent parvenir en déclarant être rattachées à un groupe qui n'est pas encore, ou ne sera plus, déclaré: leurs racines _groupe_ est **zombi**. On a fabriqué pseudo groupe dans l'arbre qui représente un groupe inconnu (temporairement ou non) de la session.

### Notes _fake_
Une note _fake_ n'a aucun contenu et a été construite pour permettre à une ou d'autres notes qui en référençaient l'id/ids de s'y rattacher. Une note _fake_ peut figurer :
- sous une racine _avatar_ réelle,
- sous une racine _groupe_ réelle (le groupe existe),
- sous une racine _groupe_ **zombi**, le groupe a existé un jour (sinon la note n'aurait pas pu être créée) mais n'existe plus, du moins dans la session en cours (résiliation ...).

Pour des raisons de lisibilité, un goupe _zombi_ a un _nomc_ :
- comme il n'existe que parce qu'une note _avatar_ y est rattachée,
- comme cette note a été créée (ou bougée) sur un groupe qui a été connu,
- la note _avatar_ a identifié son rattachement par `id/ids/nomc` ou `nomc` est celui du groupe auquel elle s'est rattachée et qui un jour ou l'autre dans le passé a bien été un nom de groupe réel (et l'est peut-être encore dans une autre session).

### Disparition d'une note
Si cette note est référencée par d'autres notes qui s'y rattache,
- elle est transformée en note _fake_,
- comme toute note _fake_ elle est attachée à une racine,
  - avatar rèelle,
  - groupe réelle,
  - ou groupe zombi.
- tout le sous-arbre depuis cette note reste attachée sur la note devenue _fake_.

### Déplacement d'une note
- **note _groupe_** : elle ne peut l'être que par rattachement à une autre note, réelle ou _fake_ du **même** groupe. Tout son sous-arbre suit.
- **note _avatar_** : elle peut être déplacée (avec son sous-arbre qui ne contient que des notes du même avatar):
  - derrière n'importe quelle note du même avatar, réelle ou _fake_,
  - directement sous son avatar,
  - derrière une note de groupe, réelle ou _fake_.
  - directement sous un groupe réel ou zombi.

### Visibilité des notes par avatar / groupe
**Avatar**
- toutes les notes portant comme id celle de l'avatar _visible_ sont visibles.

**Groupe**
- toutes les notes portant comme id celle du groupe _visible_ sont visibles.
- **mais aussi** toutes les notes d'avatar telles qu'en remontant à leur rattachements successifs on tombe sur la racine du groupe. Mais cette information ne figure **pas** dans la note qui ne mentionne que son rattachement juste supérieur. Il faut donc par transitivité remonter jusqu'à la racine.

