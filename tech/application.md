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
- selon le framework `vuejs.org (V3) + pinia`,
- et selon `quasar.dev` en tant que surcouche de `vuejs`.

L'architecture de l'application sépare:
- sa **mémoire** globale contenant:
  - des constantes de configuration,
  - les documents du compte courant, mémoires _non réactives_ et _réactives_ (stores).
- ses **vues** ayant chacune trois parties:
  - `template` : description déclarative HTML+ de ce qui apparaît à l'écran,
  - `script` : 
    - variables affichables dans le template: soit des variables de calcul local à la vue, soit _calculées_ depuis les _stores_.
    - fonctions sur ces variables.
  - `style` SASS : classes de style utilisées dans le template de la vue. 

# Configuration de l'application

## Configuration _personnalisée et buildée_
La _configuration_ peut être personnalisée (ce n'est pas du tout obligatoire), puis l'application est _buildée_, la configuration ne peut pas être modifiée après _build_.

#### Personnalisation du script `src/app/config.mjs`
Elle peut être conservée par défaut, sauf la propriété `BUILD` qui donne la date-heure du build de l'application.

#### Personnalisation des traductions dans les scripts `src/i18n/fr-FR/index.js`
Les scripts `fr-FR` et `en-EN` donnent les _traductions_ en français et en anglais.

Une personnalisation est possible mais à gérer par concaténation du script _par défaut_ avec un script de _surcharges ponctuelles_.

#### Personnalisation de l'aide en ligne `src/assets/help/...`
L'aide en ligne peut aussi être personnalisée: les fichiers d'aide _par défaut_ sont à fusionner avec des fichiers de _surcharges ponctuelles_.

> Un hébergeur procédant à une _personnalisation_ doit rendre public une liste de fichiers ayant patché les fichiers par défaut et le script de _patch_ associé afin que n'importe soit en mesure de refaire le _build_ correspondant depuis les sources _open source_ de départ et s'assurer ainsi qu'ils n'ont pas été altérés par des patchs compromettant la confidentialité.

## Configuration de _runtime_
L'application une fois _buildée_ (distribuable `[distrib]/`) est un folder comportant peu de fichiers:
- elle peut être utilisée par plusieurs hébergeurs, tous ceux ayant opté pour cette même personnalisation.
- chaque hébergeur doit écrire son propre ficher `[distrib]/etc/urls.json` avant distribution par le serveur Web statique mettant à disposition l'application.

    {
    "opurl" : "http://localhost:8443",
    "pubsuburl" : "http://localhost:8443",
    "docsurls": { "fr-FR": "http://localhost:4000/fr", "en-EN": "http://localhost:4000/en" },
    "vapid_public_key": "BC8J60JGGoZRHWJDrSbRih-0qi4Ug0LPbYsnft668oH56hqApUR0piwzZ_fsr0qGrkbOYSJ0lX1hPRTawQE88Ew",
    }

_**Remarques:**_
- ce fichier _peut_ être téléchargé / affiché par n'importe qui: il ne comporte aucune information confidentielle.
- ce fichier ne comporte _que_ les informations propres à chaque hébergeur, qui chacun a modifié la distribution sortie de _build_ pour y mettre ses propres URLs.
- `opurl`: URL du service OP de l'hébergement.
- `pubsuburl`: URL du service PUBSUB de l'hébergement (_peut_ être égal à celui de OP).
- `docsurls`: URLs dans les différentes langues supportées du site de documentation de l'application. Chaque hébergement _peut_ proposer son propre site de documentation OU choisir celui standard.
- `vapid_public_key`: chaque hébergement _peut_ avoir généré son propre couple de clés VAPID (pour le service web-push), mais peut aussi opter par facilité pour le couple de clés générées pour le test de l'application.

> Les fichiers de l'application _buildée_ sont lisible _en clair_ (à peu près !) par n'importe qui. La comparaison avec les fichiers _certifiés_ permet de déterminer si elle a subi des transformations depuis le source public _open source_.

## Chargement de la configuration: `config-store.js`
Il est assuré par le script `src/boot/appconfig.mjs` qui est lancé au boot de l'application, avant affichage de _la_ vue racine `App.vue`.

Par commodité, la configuration _compilée_ est stockée dans le _store_ `src/stores/config-store.js`: c'est peu rationnel, la configuration par principe ne devrait pas changer après chargement initial et n'aurait pas besoin d'être disponible dans un _store_ réactif. Toutefois, parce que ce sont les seules données permanentes d'une session du browser, ce _store_ inclut 5 propriétés gérées par le _service-worker_. Celles-ci évoluent donc (rarement) en cours de session: en particulier la propriété `nouvelleVersion` (effectivement réactive) est affichée dans `App.vue`.

Ce _store_ est le seul qui est constant depuis le chargement de l'application et **ne subit pas de _reset_** à la connexion à un nouveau compte.

    nouvelleVersion: false, // passe à true quand une nouvelle version est disponible
    registration: null, // objet de registration du SW
    subJSON: '???', // objet subscription obtenu de SW sérialisé
    pageSessionId: '', // identifiant universel aléatoire du chargement de la page (session browser)
    nc: 0, // numéro d'ordre de connexion dans la session du browser

Ces données sont fondamentales pour la gestion des notifications web-push:
- `subJSON` est le token web-push obtenu par le _service-worker_ depuis le browser.
- `pageSessionId` est un identifiant aléatoire représentant la session de l'application.
- `nc` est un numéro d'ordre de 1 à N de connexion croissant dans une session.

> Le couple `pageSessionId, nc` identifie universellement une session de connexion à un compte dans un browser dont l'exécution est universellement identifiée par `subJSON`.

# `service-worker` et `web-push`
`quasar` gère le _service-worker_ et met à disposition les scripts de personnalisation suivants:
- `src/pwa/register-service-worker.js` : ce script est invoqué à l'enregistrement du service-worker, donc _dans_ l'application.
- `src/pwa/custom-service-worker.js` : ce script est une extension du _service-worker_ et s'exécute donc dans le SW lui-même.

### `src/pwa/register-service-worker.js`
Le script propose des débranchements lors des évènements:
- `ready` : à cette occasion l'objet `registration` est transmis pour stockage dans `config-store`.
- `updatefound updated` : ces événements activent une action de `config-store`. La propriété (réactive) `nouvelleVersion` passe à `true` sur réception de `updated` (`updatefound` est ignoré), ce qui provoque l'affichage dans `App.vue` du bouton `Nouvelle version disponible`. 

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

Pour obtenir subJSON il faut fournir la clé publique VAPID correspondant à la clé privée employée par le service PUBSUB pour émettre les notifications web-push.

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

Ces données ne sont issues que depuis deux sources:
- **à la connexion depuis les documents lus de la base IDB** pour des sessions synchronisée ou avion.
- **depuis les documents rapportés par l'opération `Sync`**.

Dans les deux cas, les documents sont,
- _compilés_ depuis leur forme sérialisée en leur forme instance de leur classe (`src/modele.mjs`), 
- accumulés dans un _buffer_, une instance de la classe `SB` (`src/synchro.mjs`),
- puis en une seule fois, sans interruption, mémorisés dans les _stores_,
- in fine en une seule transaction stockés dans IDB (si nécessaire).

L'ensemble des _stores_ présente en conséquence toujours un état _cohérent_ (atomique) représentant exactement un état _passé_ (de peu) mais consistent des documents du périmètre du compte. Les vues à l'écran reflétant l'état _réactif_ des _stores_, elles ne font apparaître que des vues cohérentes de l'état des documents en base centrale (avec évidemment un léger différé).

> Les données des _stores_ ne sont JAMAIS mises à jour directement depuis les vues ou les opérations, elles reflètent **toujours** le résultat de la dernière opération `Sync` (`src/synchro.mjs`) effectuée.

## Données NON réactives
Ces données sont immuables après initialisation, plus exactement d'éventuelles réinitialisations donneraient les mêmes valeurs. Elles sont stockées dans de simple `Maps`.

### Registre des clés des avatars (classe : `RegCles`  - `src/modele.mjs`)
Le registre des clés délivre la clé d'un avatar depuis son id et est alimenté à chaque rencontre d'une clé d'avatar dans la compilation des documents. Comme un avatar ne change pas de clé, cette information n'a pas besoin d'être réactive.

### Registre des clés des _chats_ et _clés privées_ des avatars (classe : `RegCc`  - `src/modele.mjs`)
La clé privée d'un avatar se découvre à la compilation d'un avatar (ou du compte), sa clé immuable par principe est stockée dans ce registre.

La clé d'un _chat_ est décryptée, soit par la clé K du compte, soit par la clé RSA de l'avatar: ce registre permet d'obtenir la clé d'un _chat_ quelque soit la façon dont elle a été cryptée.

## Données _réactives_, les _stores_ : `src/stores/...`
`./stores.mjs`
- répertoire des _stores_ existant avec un getter par store.

### Données issues des _documents_ du périmètre du compte
**Rappel: le périmètre d'un compte comporte les documents suivants:**
- les singletons `espace synthese` identifiés par le `ns` de l'espace de l'organisation.
- les singletons `compte compti compta invits` portant l'identification du compte.
- un document `partition` pour les comptes O.
- tous les documents `avatar note chat sponsoring ticket` dont l'identifiant _majeur_ est celui d'un des avatars du compte.
- tous les documents `groupe note membre` dont l'identifiant _majeur_ est celui d'un des groupes où le compte est _actif_.

Tous ces documents SAUF `synthese compta partition` sont _synchronisés_: toute modification les concernant est signalée par web-push aux browsers dont une session est connectée et faisant partie de leur périmètre.
- `synthese compta partition` sont a contrario lus sur demande dans les vues qui en affichent le contenu.

`./session-store.js`
- tous les documents singletons du compte: `compte compta compti espace partition`
- les id _courantes_ `avatatarId` pour l'avatar courant, etc.
- les getters sur les états du compte.

`./avatar-store.js`
- une entrée par avatar donnant aussi ses `chats sponsorings tickets notes` (pour les notes seulement l'ids et le volume de fichier).

`./groupe-store.js`
- une entrée par groupe donnant son `chatgr`, ses `membres notes` (pour les notes seulement l'ids et le volume de fichier).
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
`./ui-store.js` : données reflétant l'état UI de la session.
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

`/src-pwa` : les fichiers pour le _service-worker_
  - `custom-service-worker.js`
  - `register-service-worker.js`

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
- de binaire à base64 et réciproquement. Identique à celui utilisé en service OP / PUBSUB afin d'éviter d'avoir deux implémentations différentes en environnement Web et node.

`config.mjs`
- élément de personnalisation de l'application et _constantes_ de configuration.

`db.mjs`
- gestion de la base IDB, lecture des tables et écritures synchronisées.

`modele.mjs`
- registres _non réactifs_ des clés.
- toutes les classes de _document_ de l'application et leur compilation depuis le format _row_ sérialisé reçu par _Sync_ et lu depuis IDB.

`net.mjs`
- fonctions d'accès à Internet GET / POST, aux services OP et PUBSUB et au _storage_.

`operation.mjs`
- classe abstraite ancêtre des _opérations_.

`operations4.mjs`
- une classe par opération.

`synchro.mjs`
- classe `Queue` : reçoit les notifications web-push, les met en queue et déclenche les opérations `Sync` correspondantes de remise à niveau des documents du périmètre.
- classe `SB` : buffer accumulant les documents à ranger en _store_ en une fois et à ranger en IDB en une transaction.
- fonction de `connexion` / `deconnexion`
- opération `Sync (SyncStd / SyncFull)` : récupération du delta des documents mis à jour sur le serveur depuis le dernier `Sync`.
- opération `GetEspace` : avec Sync, seule opération utilisée pour gérer connexion / deconnexion / synchronisation.

`util.mjs`
- diverses fonctions utilitaires.

`webcrypto.mjs`
- fonctions utilisant la cryptographie (Web-Crypto).

## Le folder `src/assets`
`./fonts`
- les fontes utilisées dans l'application

`./help`
- les ressources de l'aide en ligne (voir plus avant)

`./*.png *.jpg *.svg *.bin`
- quelques images et deux sons `beep.bin` `cliccamera.bin`

## Le folder `src/boot`
Scripts exécutés au boot:
- `appconfig.js` : chargement de la configuration.
- `axios.js i18n.js` : pour _quasar_.

## Le folder `src/css`
- fichiers de style `css` et `sass`, définissant des valeurs par défaut et des personnalisations à importer sur demande dans les sections `<style>` des vues.

## Le folder `src/i18n`
Un sous folder par langue traduite dont le fichier `index.js` exporte tous les termes traduits.

## Le folder `src/router`
Inutilisé, à laisser tel quel.

# Les "vues"

## Structure des `*.vue`
La structure actuellement préconisée offre des possibilités nouvelles par rapport à _l'historique_ (comme la possibilité d'avoir des `await` dans le `setup`) et quelques simplifications / clarifications.

Trois sections: `<template> <script setup> <style>`

    <template>
      <div .../>
      ...
      </div>
    </template>

    <script setup>
      import { ref, computed, watch, onUnmounted } from 'vue'
      import { edvol, $t } from '../app/util.mjs'
      
      import Foo from './Foo.vue'
      import Bar from './Bar.vue'

      const props = defineProps({ foo: String }) // accessible par props.foo
      const model = defineModel({ type: Object }) // la propriété "v-model"
      const emit = defineEmits(['change', 'delete'])
      const var1 = ref() // accessible par var1.value
      const var2 = computed(() => { ...}) // accessible par var2.value
      function f1 (x) { ... }
    </script>

    <style>
    ...
    </style>

Voir le détail ici : https://vuejs.org/api/sfc-script-setup.html

### Variables déclarées `ref` versus `computed`
Dans une vue on peut afficher / traiter:
- **des variables de _store_** déclarées et gérées dans un _store_ en tant que,
  - a) getters (éventuellement avec des paramètres ce qui est à peu près une _action_), 
  - b) actions. Ceci correspond à un état _global_ de la session, indépendant de toute vue.
- **des variables _locales_ à la vue** déclarées en `ref()` ou en `computed()` dans le _script setup_.

### Variables de _store_
Elles se déclarent par `computed()`

    const avatar = computed(() => aSt.getAvatar(props.id)) 

et s'utilise ensuite par `avatar.value`

#### ref() NE RÉPERCUTE PAS la réactivité
Dans l'exemple précédent si on écrit:

    const props = defineProps({ chatc: Object, ...})
    const aSt = stores.avatar
    const chatX = ref(aSt.getChat(chatc.value.id, chatc.value.ids))

`chatX` n'est PAS réactif et contient une valeur initiale (qui peut être changée par `chatx.value = ...`): quand le _store_ évolue, `chatX` reste inchangé.

Pour que `chatx` soit _réactif_ aux changements du _store_, il aurait fallu écrire:

    const chatX = computed(() => aSt.getChat(chatc.value.id, chatc.value.ids))

**`ref()` rend réactive une variable locale mais ne transmet pas la réactivité de l'expression qui l'a initialisée.**

Les variables déclarées comme `const v = computed(() => {...})` sont _calculées_, leur contenu `v.value` est lisible mais ne peut pas être changé par affectation.

## `src/App.vue`
C'est LE layout unique décrivant LA page de l'application. Elle est constituée des éléments suivants:
- **headaer**
  - _boutons à gauche_: aide, notifications, menu, accueil, page _back_, statut de la session,fichiers visibles en avion, presse-papier
  - _titre de la page courante_. Le cas échéant une seconde barre affiche les onglets pour les pages ayant des onglets.
  - _boutons à droite_: ouverture du drawer de recherche (si la page a une recherche), aide.
- **footer**:
  - _boutons à gauche_: langue, mode clair / foncé, outils,
  - _information du compte connecté_ son type `D/A/O`, son nom, son organisation,
  - _bouton de déconnexion_.
- **drawer de filtre à droite** affichant les filtres de recherche pour les pages en ayant. 
  - il s'affiche par appui sur le bouton de recherche (en haut à droite).
  - quand la page est assez large, le drawer de filtre reste affiché à côté de la page principale, sinon sur la page principale qui en est partiellement recouverte.
  - pour chaque page ayant un filtre, la liste des composants constituant le filtre est affichée.
- **container de la page principale**: 
  - il contient à un instant donné une des _pages_ listées dans la section "Pages". 
  - celles-ci sont formées par un tag `<q-page>` qui s'intègre dans le tag `<q-page-container>` de App.

**App inclut quelques dialogues singletons** afin d'éviter leurs inclusions trop multiples:
- ces dialogues n'ont pas de propriétés, c'est le contexte courant qui fixe ce qu'ils doivent afficher.
- chaque dialogue dans App est gardé par un `v-if` de la variable modèle qui l'ouvre.
- `DialogueErreur DialogueHelp PressePapier PanelPeople OutilsTests PhraseSecrete`

**App a quelques dialogues internes simples:**
- _ui.d.a.aunmessage_ : Gestion d'un message s'affichant en bas
- _ui.d.a.diag_ : Affiche d'un message demandant confirmation 'j'ai lu'
- -ui.d.a.estzombi_ : Affiche l'annonce de suppression proche du compte
- _ui.d.a.confirmFerm_ : Demande de confirmation d'une fermeture de dialogue avec perte de saisie
- _ui.d.a.reload_ : Information / option d'installation d'une nouvelle version
- _ui.d.a.dialoguedrc_ : Choix de déconnexion, reconnexion, continuer
- _ui.d.a.opDialog_ : Affiche l'opération en cours et propose son interruption
- _ui.d.a.confirmstopop_ : Confirmation d'interruption de l'opération en cours
- _ui.d.a.sync_ : Gestion de la synchronisation automatique

La logique embarquée se limite à:
- détecter le changement de largeur de la page pour faire gérer correctement l'ouverture du drawer de filtre dans `stores.ui`,
- gérer le titre des pages,
- se débrancher vers les pages demandées.

## Les _pages_ : `src/pages/Page...vue`
Une page s'installe dans `App.vue` dans le `<page-container>` et occupe tout l'espace (sauf _header_ et _footer_):
- il y a TOUJOURS une page courante affichée (`ui.page`).
- on change de page par `ui.setPage('NouvellePage')`
  - la nouvelle page est affichée.
  - si la page actuelle est déclarée dans `ui` comme `pageB` (`espace compte groupes groupesac`), quand on appuie sur le bouton _back_, ça renvoie à cette page. Ceci permet par exemple de revenir à la liste des groupes quand on est sur la page d'un groupe en venant de la liste. `
- au changement de page, tous les dialogues ouverts par la page sont automatiquement fermés (voir plus avant).

Une _page_ peut importer des _components_ et importer / ouvrir des _dialogues_:
- ses propres dialogues.
- des _panels_.
- des _dialogues_.

Voir en annexe la liste des **pages**.

## Les _panels_ : `src/panels/...vue`
Un _panel_ est techniquement un `<q-dialog>`:
- il occupe presque toute la hauteur et a une largeur importante.
- il se découvre depuis la gauche et se replie vers la gauche quand il est fermé par son bouton de fermeture en haut à gauche.
- en général il correspond à un dialogue de saisie complexe mais peut aussi n'être qu'un affichage volumineux (par exemple un _chat_).

Un _panel_ peut importer des _components_ et importer / ouvrir des _dialogues_:
- ses propres dialogues.
- des _dialogues_.

Voir en annexe la liste des **panels**.

## Les _dialogues_ : `src/dialogues/...vue`
Un _dialogue_ est techniquement un `<q-dialog>`:
- il occupe une place centrale, mais limitée, dans la page.
- il se découvre progressivement et se cache progressivement (_fade in / out_) quand il est fermé par son bouton de fermeture en haut à gauche.
- en général il correspond à un dialogue de saisie.

Un _dialogue_ peut importer des _components_ et importer / ouvrir ses propres dialogues.

Voir en annexe la liste des **dialogues**.

## Les _components_ : `src/components/...vue`
Ils sont importés et inclus dans les autres éléments.

Ils peuvent importer d'autres _components_.

Voir en annexe la liste des **components**.

## Maîtrise des cycles d'importation
Un cycle (le composant A inclut le composant B qui inclut composant A) est une faute de conception que Webpack détecte (il ne sait pas comment faire) mais n'indique malheureusement pas clairement.

Pour chaque composant un numéro de couche est géré:
- les composants n'important rien ont pour numéro de couche 1.
- un composant a pour numéro de couche le plus haut des numéros de couche des composants importés + 1.
- la feuille Excel `Dépendances.xls` en tient attachement ce qui permet de s'assurer qu'un cycle n'a pas fortuitement été introduit dans la conception.

## Gestion des dialogues
Les objectifs de cette gestion sont:
- de gérer une pile des dialogues ouverts par une page afin de pouvoir les fermer sans avoir à se rappeler de leur empilement éventuel,
- de pouvoir fermer simplement le dernier dialogue ouvert sans avoir à le citer.

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

# Styles clairs et foncés
Le style global peut être clair ou foncé selon la variable de Quasar `$q.dark.isActive`

Quand il y a des listes à afficher, il est souhaitable d'afficher une ligne sur deux avec un fond légèrement différent, donc avec un style dépendant de l'index `idx` de l'item dans la liste qui le contient. D'où les classes suivantes:
- `sombre sombre0`: fond très sombre, fonte blanche pour les idx pairs (ou absents).
- `sombre1`: fond un peu moins sombre, fonte blanche pour les idx impairs.
- `clair clair0`: fond très clair, fonte noire pour les idx pairs (ou absents).
- `clair1`: fond un peu moins clair, fonte noire pour les idx impairs.

Dans `util.js` les fonctions suivantes fixent dynamiquement le fond à appliquer selon que le mode sombre est activé ou non et l'index idx éventuel:
- `dkli (idx)` : fond _dark_ ou _light_ selon idx
- `sty ()` : fond _dark_ ou _light_
- `styp (size)` : pour un dialogue, fond _dark_ ou _light_, largeur fixée par size (`'sm' 'md' 'lg'`) et ombre claire ou foncée.

### L'affichage MarkDown par VueShowdown
Le component `vue-showdown` affiche le contenu d'un texte MD dans un `<div>`. 

Sa classe de style principal `markdown-body` porte un nom **fixe** de manière assez contraignante (ne supporte pas un nom de classe dynamique). Ceci oblige à un avoir un component distinct pour chaque style désiré:
- `SdBlanc [texte]`: la fonte du texte est blanche (pour des fonds foncés).
- `SdNoir [texte]`: la fonte du texte est noire (pour des fonds clairs).
- `SdRouge [texte]`: la fonte du texte est rouge (pour des fonds clairs ou foncés).

Dans ces components le fond N'EST PAS fixé, il est transparent, et suivra celui de l'environnement. Mais celui de la fonte doit l'être, d'où le component suivant:
- `SdNb [texte idx]`: il choisit entre `SdBlanc` et `SdNoir` selon, a) que le mode Quasar _dark_ est actif ou non, b) que idx passé en propriété est absent ou pair ou impair (??? pas clair, à vérifier).

> Remarque: de facto seul `SdNb` est utilisé dans les autres éléments. Et encore car l'affichage d'un MD s'effectue quasiment tout le temps par le component `ShowHtml` qui englobe `SdNb`. Toutefois il existe des cas ponctuels d'utilisation de SdBlanc et SdRouge.

# Opérations
Toutes les opérations sont rassemblées dans `src/app/operations4.mjs`, sauf `Sync (SyncStd / SyncFull)` et `GetEspace` qui sont dans `src/app/synchro.mjs`:
- les opérations standard héritent de Operation (src/app/operation.mjs), 
- les opérations `Sync (SyncStd / SyncFull)` héritent de `OperationS` (`src/app/synchro.mjs`) qui hérite de `Operation`.
- une opération a un constructeur acceptant son _code_ en paramètre. Le _code_ est une entrée de traduction dans i18n.
- une opération a une méthode async run ayant des paramètres spécifiques de l'opération.
  - elle peut retourner un résultat et se termine par appel de la méthode générique finOK().
  - elle trappe les exceptions et les transmet à la méthode générique finKO().

  export class MonOp extends Operation {
    constructor() { super('MonOp') }

    async run (...) {
      ...
      this.finOK() // OU return this.finOK(ret)
    } catch (e) {
      await this.finKO(e)
    }
  }

Une opération _standard_ peut être interrompue par l'utilisateur et sortir en exception.

### Sortie en Exception
#### Les exceptions _tueuses_
- ce sont les exceptions de code compris entre 8990 et 9000.
- pour les opérations Sync, **toutes** les exceptions sont _tueuses_.

Une exception _tueuse_ affiche la page `clos` qui ne laisse aucune autre solution que de sortir de l'application.

Les exceptions _normales_:
- provoquent l'ouverture du dialogue `DialogueErreur.vue`
- la sortie de ce dialogue a les options suivantes, selon le type d'exception et le choix de l'utilisateur:
  - **se déconnecter**,
  - **se déconnecter et tenter de se reconnecter** au même compte avec la même phrase secrète.,
  - **ne rien faire**, c'est à dire laisser la session se poursuivre _comme si_ l'opération n'avait pas été lancée. Dans ce cas on peut récupérer cette exception dans l'application (APRES son passage par `DialogueErreur.vue`).

  try {
    await new ErreurFonc().run('Mon erreur', 1)
  } catch (e) {
    console.log(e.toString())
  }

# Synchronisation

## Fin d'opération de mise à jour de OP `notif`
Lorsqu'une opération de mise à jour s'exécute dans OP, un certain nombre de documents sont mis à jour, leur version a changé: un objet `trlog` est créé.
- cet objet a une forme _longue_ qui est transmise à PUBSUB sur la méthode / requête `notif`.
  - le traitement par PUBSUB a une première phase _synchrone_ qui,
    - met à jour l'état mémoire des sessions,
    - prépare la liste des messages de notifications à envoyer: chaque message a pour structure un `trlog` de forme raccourcie.
  - la second phase est asynchrone et consiste à émettre tous les messages de notification préparés en phase 1.
  - la méthode / requête `notif` est courte vu du côté de l'appelant OP et ne diffère que de peu le retour de l'opération de mise à jour.
- l'objet `trlog` a une forme raccourcie quand il parvient dans les sessions:
  - la session appelante de l'opération: les mises à jour ayant concerné au moins un document du périmètre du compte (sauf exception ?).
  - les autres sessions enregistrées par PUBSUB _impactées_ c'est à dire ayant au moins un des documents de leur périmètre mis à jour par l'opération (possiblement aucune session). Chaque session recevra en message de notification un `trlog` raccourci.

En session on peut ainsi recevoir des `trlog` depuis deux sources:
- en résultat d'une opération de mise à jour soumise par la session elle-même,
- par suite d'une opération de mise à jour déclenchée par une autre session et parvenue en _notification_.

Le traitement est géré par `syncQueue.synchro(trlog)` (ou `syncQueue` est le singleton de la classe `Queue` `src/app/synchro.mjs`):
- cette méthode empile les `trlog` reçus (les met en _queue_) en ignorant ceux dont la version est plus ancienne que la dernière prise en compte.
- s'il y a un traitement de Sync en cours, la méthode se limite à cette mise en attente.
- sinon la méthode invoque en asynchrone un traitement de `sync` qui va demander les données correspondantes aux avis de changements reçus. 

### Objet `trlog`
- `sessionId`: `rnd.nc`. Permet de s'assurer que ce n'est pas une notification obsolète d'une connexion antérieure.
- `partId`: ID de la partition si c'est un compte "0", sinon ''.
- `vpa`: version de cette partition ou 0 si inchangée ou absente.
- `vce`: version du compte. (utile ?)
- `vci`: version du document `compti` s'il a changé, sinon 0.
- `lavgr`: liste `[ [idi, vi], ...]` des Couples des IDs des avatars et groupes ayant été impactés avec leur version.
- `lper`: **format long seulement**. `liste [ {...}, ...]` des `perimetre` des comptes ayant été impactés par l'opération (sauf celui de l'opération initiatrice).

## L'objet `DataSync`
Cet objet sert:
- entre session et _service OP_ a obtenir les documents resynchronisant la session avec l'état de la base.
- dans une base locale IDB: à indiquer ce qui y est stocké et dans quelle version.

**Les états successifs _de la base_ sont toujours cohérents**: _tous_ les documents de _chaque_ périmètre d'un compte sont cohérents entre eux.

L'état courant d'une session en mémoire et le cas échéant de sa base locale IDB, est consigné dans l'objet `DataSync` ci-dessous:
- chaque sous-arbre d'un avatar ou d'un groupe est _cohérent_ (tous les documents sont synchrones sur la même version `vs`),
- en revanche il peut y avoir (plus ou moins temporairement) des sous-arbres à jour par rapport à la base et d'autres en retard.

**L'objet `DataSync`:** (`src/app/api.mjs`)
- `compte`: `{ vs, vb }`
  - `vs` : numéro de version de l'image détenue en session
  - `vb` : numéro de version de l'image en base centrale
- `avatars`: Map des avatars du périmètre. 
  - Clé: id de l'avatar
  - Valeur: `{ id, chg, vs, vb } `
    - `chg`: true si l'avatar a été détecté changé en base par le serveur
- `groupes`: : Map des groupes du périmètre. 
  - Clé: id groupe
  - Valeur: `{ id, chg, vs, vb, ms, ns, m, n }`
    - `chg`: true si le groupe a été détecté changé en base par le serveur
    - `vs` : numéro de version du sous-arbre détenue en session
    - `vb` : numéro de version du sous-arbre en base centrale
    - `ms` : true si la session a la liste des membres
    - `ns` : true si la session a la liste des notes
    - `m` : true si en base centrale le groupe indique que le compte a accès aux membres
    - `n` : true si en base centrale le groupe indique que le compte a accès aux membres

**Remarques:**
- un `DataSync` reflète l'état d'une session, les `vs` (et `ms ns` des groupes) indiquent quelles versions sont connues d'une session.
- Un `DataSync` reflète aussi l'état en base centrale, du moins quand il a été écrit, les `vb` (et `m n` pour les groupes) indiquent quelles versions sont détenues dans l'état courant de la base centrale.
- Quand toutes les `vb` et `vs` correspondantes sont égales (et les couples `ms ns / m n` pour les groupes), l'état en session reflète celui en base centrale: il n'y a plus rien à synchroniser ... jusqu'à ce l'état en base centrale change et que l'existence d'une mise à jour soit notifiée par _web push_ à la session.

Chaque appel de l'opération `Sync` :
- transmet le `DataSync` donnant l'image connue en session,
- reçoit en retour,
  - le `DataSync` rafraîchi par le dernier état courant en base centrale.
  - le dernier état, s'il a changé, des documents `comptes comptis invits` du compte,
  - zéro, un ou plusieurs lots de mises de sous-arbres _avatar_ et _groupe_ entiers.

Pas forcément les mises à jour de **tous** les sous-arbres:
- le volume pourrait être trop important,
- le nombre de sous-arbres mis à jour dépend du volume de la mise à jour.
- en conséquence si le `DataSync` indique que tous n'ont pas été transmis, une opération `Sync` est relancée avec le dernier `DataSync` reçu.

**Cas particulier de la connexion,** premier appel de `Sync` de la session:
- c'est le serveur qui construit le `DataSync` depuis l'état du compte et les versions des sous-arbres **qu'il va tous chercher**.
- au retour, la session va récupérer (en mode _synchronisé_) le `maxim` de documents encore valides et présents dans IDB: 
  - elle lit depuis IDB le `DataSync` qui était valide lors de la fin de la session précédente et qui donne les versions `vs` (et `ms ns` pour les groupes),
  - elle lit depuis IDB l'état des sous-arbres connus afin d'éviter un rechargement total: les `vs` (et `ms ns`) sont mis à jour dans le DataSync.
  - le prochain appel de `Sync` ne provoquera des chargements _que_ de ce qui est nouveau et pas des documents ayant une version déjà à jour en session UI.

**Appels suivants de Sync**
- le `DataSync` reçu sur le serveur permet de savoir ce que la session connaît.
- si des avis de mises à jour sont parvenus, la liste de leur `id` est passée à `Sync`: au lieu de relire toutes les versions de tous les sous-arbres `Sync` se contente de lire uniquement les versions des sous-arbres changés donnés par la liste des `id` reçue de la session UI.

A chaque appel de `Sync`, les versions de` comptes comptis invits` sont vérifiées: en effet avant de transmettre les mises à jour des sous-arbres `Sync` s'enquiert auprès du document comptes:
- des sous-arbres n'ayant plus d'intérêt (avatars et groupes hors périmètre),
- des nouveaux sous-arbres (nouveaux avatars, nouveau groupes apparaissant dans le périmètre),
- pour les groupes si les accès _membres_ et _notes_ ont changé pour le compte.

Voir dans src/app/synchro.mjs les opérations de `Connexion...` et `Sync...`.

## Synchronisation _automatique_ versus _sur demande_
Par défaut la synchronisation est automatique, les avis de changements des documents par les opérations des **autres** sessions sont reçus par web-push et traités.

MAIS le service PUBSUB peut s'interrompre sans que le service OP ne soit interrompu:
- les opérations peuvent toujours être effectuées MAIS la session ne reçoit plus les avis des autres sessions: son affichage est retardé.

En _synchronisation sur demande_, sauf demande explicite de l'utilisateur quand le service PUBSUB est down, l'utilisateur déclenche une synchronisation forcée par appui sur un bouton.
- mais le _heartbeat_ qui indique au service PUBSUB que la session est toujours vivante ne fonctionne plus.
- l'utilisateur peut relancer ce _heartbeat_, du moins tenter de le faire: si le service PUBSUB est _up_ la synchronisation redevient _automatique_.

## Gestion du _heartbeat_
Elle est assurée dans `session-store` par les actions `startHB` et `stopHB`:

`startHB` émet périodiquement une requête POST au service PUBSUB en incrémentant le numéro d'envoi afin de pouvoir détecter le cas échéant une rupture dans la séquence des web-push d'avis de modification des documents du périmètre de la session.

La déconnexion poste aussi un avis au service PUBSUB pour l'informer de la fin de la session et lui permettre de supprimer les données de la session qu'il conserve.

# Base locale IDB

(TODO)

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

## `localStorage` et IDB
**En mode *avion*** dans le `localStorage` les clés `monorg-hhh` donne chacune le numéro de compte `ccc` associé à la phrase de connexion dont le hash est `hhh` : `monorg-ccc` est le nom de la base IDB qui contient les données de la session de ce compte pour l'organisation `monorg` dans ce browser.

**En mode *synchronisé***, il se peut que la phrase secrète actuelle enregistrée dans le serveur (dont le hash est `hhh`) ait changé depuis la dernière session synchronisée exécutée pour ce compte :
- si la clé `monorg-hhh` n'existe pas : elle est créée avec pour valeur `monorg-ccc` (le nom de la base pour le compte `ccc`).
- si la base `monorg-ccc` n'existe pas elle est créée.
- l'ancienne clé, désormais obsolète, pointe bien vers le même compte mais ne permet plus d'accéder à ce compte, dont la clé K a été ré-encryptée par la nouvelle phrase.

#### La table `sessionsync`
Cette table enregistre les date-heures,
- de la session synchronisée précédente correctement connectée puis terminée : `dhdebut dhfin`
- de la session synchronisée en cours : 
  - `dhlogin` : dh de fin de login,
  - `dhsync` : date-heure de fin de la dernière opération de synchronisation,
  - `dhpong` : date-heure de réception du dernier _pong_ reçu sur le websocket attestant que celui-ci n'est pas déconnecté.

Cet objet est disponible dans store/db `sessionsync`, uniquement quand la session courante est _synchronisée_.

# Démon _heartbeat_ et ficavion
(TODO)

En mode synchronisé un démon tourne en tâche de fond pour obtenir du serveur les fichiers requis pas encore disponibles en IDB

# Annexe: liste des _pages_

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

Dialogue:
- PNdl: dialogue gérant le chargement des notes en local.

### PageFicavion (2)
Affiche la liste des fichiers visible en mode avion et pour chacun,
- permet de l'afficher et de l'enregistrer localement,
- de voir la note à laquelle il est attaché.

# Annexe: liste des _panels_

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

# Annexe: liste des _dialogues_
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

# Annexe: _components_ particuliers
### Les filtres
La page principale App a un drawer à droite réservé à afficher les filtres de sélection propres à chaque page et permettant de restreindre la liste des éléments à afficher dans la page (par exemple les notes).

Chaque filtre est un component simple de saisie d'une seule donnée: la valeur filtrée étant stockée en store.

- **FiltreNom**: saisie d'un texte filtrant le début d'un nom ou un texte .contenu dans un string.
- **FiltreMc**: liste de mots clés (qui peuvent être soit requis, soit interdits).
- **FiltreNbj**: saisie d'un nombre jours 1, 7, 30, 90, 9999.
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
- **FiltreRac**: menu de sélection d'un statut de chat:  
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
- **FiltreTri**: sélectionne un critère de tri dans une des deux listes TRIespace et TRItranche définies au dictionnaire. 
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

# Annexe: l'arbre des notes

(TODO)

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

