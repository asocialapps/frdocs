---
layout: page
title: L'application Web
---

L'application est une page Web, _PWA: progressive web app_:
- elle n'a qu'une page (au sens Web),
- un script _service worker_ assure la disponibilité de la page en l'absence de réseau,
- l'application **reçoit** des notifications web-push transmises par le browser qui les a reçues du service PUBSUB afin de notifier une évolution des documents du périmètre du compte connecté.
- l'application **émet** des requêtes HTTP:
  - vers le _service OP_, opérations de mises à jour / consultations / synchronisation,
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
La _configuration_ peut être personnalisée (ce n'est pas obligatoire), puis l'application est _buildée_: la configuration ne peut pas être modifiée après _build_.

#### Personnalisation du script `src/app/config.mjs`
Elle peut être conservée par défaut, sauf la propriété `BUILD` qui donne une référence du build de l'application.

#### Personnalisation des traductions dans les scripts `src/i18n/fr-FR/index.js`
Les scripts `fr-FR` et `en-EN` donnent les _traductions_ en français et en anglais.

> Une personnalisation est possible mais à gérer par concaténation du script _par défaut_ avec un script de _surcharges ponctuelles_.

#### Personnalisation de l'aide en ligne `src/assets/help/...`
L'aide en ligne peut aussi être personnalisée: les fichiers d'aide _par défaut_ sont à fusionner avec des fichiers de _surcharges ponctuelles_.

> Un hébergeur procédant à une _personnalisation_ doit rendre public une liste de fichiers ayant patché les fichiers par défaut et le script de _patch_ associé afin que n'importe qui soit en mesure de refaire le _build_ correspondant depuis les sources _open source_ de départ et s'assurer ainsi qu'ils n'ont pas été altérés par des patchs compromettant la confidentialité.

## Configuration de _runtime_
L'application une fois _buildée_ (distribuable `dist/pwa`) est un folder comportant peu de fichiers:
- elle peut être utilisée par plusieurs hébergeurs, tous ceux ayant opté pour cette même personnalisation.
- chaque hébergeur doit corriger les deux fichiers suivants par les siens avant distribution par le serveur Web statique mettant à disposition l'application:
  - `dist/pwa/services.json`: décrit vers quels URls les services sont assurés en fonction des organisations.
  - `dist/pwa/README.md` : information des utilisateurs sur la build de l'application qu'il utilise.
  - `dist/pwa/patchs.zip` : fichiers sources éventuels utilisés pour _patcher_ la distribution _open source_.
    - un `README.md` exprime comment les installer.
    - un ou des scripts automatise l'installation.
    - ce fichier permet à n'importe qui de _refaire une distribution / build_ depuis les sources du dépôt git `asocial-app` (open source).

Exemple de `services.json`

    {
      "vapid_public_key": "BC8J60J...QE88Ew",
      "docsurls": { 
        "fr-FR": "https://asocialapps.github.io/frdocs", 
        "en-EN": "https://asocialapps.github.io/frdocs" 
      },
      "services": {
        "a": {
          "opurl": "https://....com",
          "pubsuburl": "https://....com",
          "orgs": ["demo", "monorg"]
        },
        "z": {
          "opurl": "http://localhost:8080",
          "pubsuburl": "http://localhost:8080",
          "orgs": []        
        }
      } 
    }

_**Remarques:**_
- ce fichier _peut_ être téléchargé / affiché par n'importe qui: il ne comporte aucune information confidentielle.
- ce fichier ne comporte _que_ les informations propres à chaque hébergeur, qui chacun a modifié la distribution sortie de _build_ pour y mettre ses propres URLs.
- `vapid_public_key`: chaque hébergement _peut_ avoir généré son propre couple de clés VAPID (pour le service web-push), mais peut aussi opter par facilité pour le couple de clés générées pour le test de l'application.
- `docsurls`: URLs dans les différentes langues supportées du site de documentation de l'application. Chaque hébergement _peut_ proposer son propre site de documentation OU choisir celui standard ci-dessus.
- sous la rubrique `services`, il y a un groupe d'information par _serveur_ (en fait couple de services OP / PUBSUB) indiquant quelles organisations sont servies par ce _serveur_.
  - `opurl`: URL du service OP de l'hébergement.
  - `pubsuburl`: URL du service PUBSUB de l'hébergement (_peut_ être égal à celui de OP).
  - `"orgs": ["org1", "org2", ...]` : liste des organisations servies par ce serveur.

**Exemple de `README.md`**

    Application "asocial":
    - distribution: t1
    - build: v1.3.4 - 1.01
    - url: https://asocialapps.github.io/t1

    Sources: 
    - dépôt: https://github.com/dsportes/asocial-app/tree/v1.3.4
    - tag: v1.3.4
    - patchs: _aucun_

Ce README permet à n'importe qui de reconstituer l'application Web distribuée depuis les sources:
- le dépôt des sources est cité ainsi que la tag qui pointe exactement la révision utilisée.
- la section `patchs` résume ce qui a été modifié dans ces sources avant build. 
  - les fichiers correspondants sont à stocker dans `patchs.zip`.
  - un README.md exprime comment les installer.
  - un ou des scripts automatise l'installation.

Pour ce connecter, un compte d'une organisation va indiquer le couple _org1 / phrase secrète_ de son compte. 
- le _pseudo_ compte `admin` doit être indiqué sous la forme `admin-a` ou `admin-z` c'est à dire en précisant le serveur.
- le service `"z"` n'est à décrire que si on veut utiliser l'application Web pour tester un serveur local (sur `localhost`): dans ce cas il faut spécifier `org1-z` au lieu de `org1` comme code d'organisation afin d'être dirigé sur le serveur de test et non de production.

> Les quelques fichiers de l'application _distribuée_ sont lisibles _en clair_ (à peu près, dans un browser en debug) par n'importe qui. 

> Il est possible de les comparer avec ceux du dépôt public _open source_, patchés le cas échéant comme indiqué dans le `README.md`, après un _build_. On peut ainsi s'assurer qu'aucune intervention malicieuse cachée n'a été appliquée dans la livraison.

## Chargement de la configuration: `config-store.js`
Il est assuré par le script `src/boot/appconfig.mjs` qui est lancé au boot de l'application, avant affichage de _la_ vue racine `App.vue`.

Par commodité, la configuration _compilée_ est stockée dans le _store_ `src/stores/config-store.js`: c'est peu rationnel, la configuration par principe ne devrait pas changer après chargement initial et n'aurait pas besoin d'être disponible dans un _store_ réactif. 

**Ce sont les seules données permanentes d'une session du browser:** ce _store_ inclut des propriétés gérées par le _service-worker_. Celles-ci évoluent donc (rarement) en cours de session: en particulier la propriété `nouvelleVersion` (effectivement réactive) est affichée dans `App.vue`.

Ce _store_ est le seul qui est constant depuis le chargement de l'application et **ne subit pas de _reset_** à la connexion à un nouveau compte.

    nouvelleVersion: false, // passe à true quand une nouvelle version est disponible
    registration: null, // objet de registration du SW
    subJSON: '???', // objet subscription obtenu de SW sérialisé
    pageSessionId: '', // identifiant universel aléatoire du chargement de la page (session browser)
    nc: 0, // numéro d'ordre de connexion dans la session du browser
    permState: '???', // granted denied prompt

Ces données sont utilisées pour la gestion des notifications web-push:
- `subJSON` est le token web-push obtenu par le _service-worker_ depuis le browser.
- `pageSessionId` est un identifiant aléatoire représentant la session de l'application.
- `nc` est un numéro d'ordre de 1 à N de connexion croissant dans une session.

> Le couple `pageSessionId, nc` identifie universellement une session de connexion à un compte dans un browser dont l'exécution est universellement identifiée par `subJSON`.

# `service-worker` et `web-push`
`quasar` gère le _service-worker_ et met à disposition les scripts de personnalisation suivants:
- `src/pwa/register-service-worker.js` : ce script est invoqué à l'enregistrement du service-worker, donc _dans_ l'application.
- `src/pwa/custom-service-worker.js` : ce script est une extension du _service-worker_ et s'exécute donc dans le SW lui-même.

### `src/pwa/register-service-worker.js`
Ce script propose des débranchements lors des évènements:
- `ready` : à cette occasion l'objet `registration` est transmis pour stockage dans `config-store`.
- `updatefound updated` : ces événements activent une action de `config-store`. La propriété (réactive) `nouvelleVersion` passe à `true` sur réception de `updated` (`updatefound` est ignoré), ce qui provoque l'affichage dans `App.vue` du bouton `Nouvelle version disponible`. 

> Mis à part l'affichage d'une alerte quand une nouvelle version est disponible, l'application ne gère pas son remplacement effectif effectué automatiquement par `quasar`.

### Recevoir les notifications web-push
Il faut d'abord indiquer au browser que l'application est à l'écoute de celles-ci:
- quand le SW est `ready` il transmet sur l'évènement qui indique cet état un objet `registration` qui représente l'enregistrement du SW. Cet objet est conservé dans `config-store.regisration`.
- `permState` est le statut d'acceptation des notifications dans le browser pour l'application:
  - `prompt`: l'utilisateur ne s'est pas encore prononcé ou a réinitialisé les permissions,
  - `granted`: l'utilisateur a accepté,
  - `denied`: l'utilisateur a refusé.
  - on écoute les changements de ce statut en permanence, l'utilisateur pouvant agir à n'importe quel instant en dehors de tout contrôle de l'application. Quand le statut passe à `granted` on peut récupérer la `subscription`.
- on obtient le jeton du browser depuis cet objet `registration`:

Extrait de config-store:

    async setRegistration(registration) {
      await this.listenPerm()
      this.registration = registration
      if (this.permState === 'granted') await this.setSubscription()
      console.log('SW ready. subJSON: ' + this.subJSON.substring(0, 50))
    },

    async listenPerm () {
      await this.getPerm()
      this.notificationPerm.onchange = async () => {
        console.log("User decided to change his settings. New permission: " + this.notificationPerm.state)
        await this.getPerm()
        if (this.permState === 'granted') await this.setSubscription()
      }
    },

    async setSubscription () {
      if (!this.registration) return
      try {
        let subscription = await this.registration.pushManager.getSubscription() // déjà faite
        if (!subscription) subscription = await this.registration.pushManager.subscribe({
            userVisibleOnly: true,
            applicationServerKey: b64ToU8(this.vapid_public_key)
          })
        this.subJSON = JSON.stringify(subscription)
      } catch (e) {
        this.subJSON = '???' + e.message
      }
    },

    // ServiceWorker : événements de détection de changement de version
    setSwev (x) {
      console.log('SW event reçu:', x)
      if (x === 'updated') this.nouvelleVersion = true
    }

Pour obtenir `subJSON` il faut fournir la clé publique VAPID correspondant à la clé privée employée par le service PUBSUB pour émettre les notifications web-push.

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

    // Dans appconfig.js : lecture de la configuartion:
    new BroadcastChannel('channel-pubsub').onmessage = msgPush

    // fonction déclarée dans ce fichier
    async function msgPush (event) {
      if (event.data && event.data.type === 'pubsub') {
        try {
          const obj = decode(b64ToU8(event.data.payload))
          if (obj.sessionId === stores.session.sessionId)
            syncQueue.synchro(obj.trLog)
        } catch (e) {
          console.log('msgPush: ' + e.toString())
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

## Déconnexions sur _inactivité_
Afin d'éviter qu'une session inactive continue à envoyer des _heartbeats_ au service PUBSUB le maintenant inutilement en vie, l'activité des sessions est détectée à l'occasion:
- d'un appui sur un _bouton_,
- d'un changement de _page_ ou d'onglet sur une page affichée.

En modes _synchronisé / incognito_, à l'occasion de l'envoi d'un _heartbeat_, si la session n'a pas reçu un signe de vie depuis 10 minutes, le _heartbeat_ n'est pas envoyé et la session est déconnectée (revient à la page de login).

# Données d'une session connectée
Toutes les données sont effacées au début d'une session connectée, il ne reste rien des sessions antérieures (à l'exception de `config-store`).

Ces données ne sont issues que depuis deux sources:
- **à la connexion depuis les documents lus de la base IDB** pour des sessions synchronisée ou avion.
- **depuis les documents rapportés par l'opération `Sync`**.

Dans les deux cas, les documents sont,
- _compilés_ depuis leur forme sérialisée en leur forme instance de leur classe (`src/modele.mjs`), 
- accumulés dans un _buffer_, une instance de la classe `SB` (`src/synchro.mjs`),
- puis en une seule fois, sans interruption, mémorisés dans les _stores_,
- in fine en une seule transaction stockés dans IDB (en mode _synchronisé_).

L'ensemble des _stores_ présente en conséquence toujours un état _cohérent_ (atomique) représentant exactement un état _passé_ (de peu) mais consistent des documents du périmètre du compte. 

Les vues à l'écran reflétant l'état _réactif_ des _stores_, elles ne font apparaître que des vues cohérentes de l'état des documents en base centrale (avec évidemment un léger différé).

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
- répertoire des _stores_ existant avec un _getter_ par store.

### Données issues des _documents_ du périmètre du compte
**Rappel: le périmètre d'un compte comporte les documents suivants:**
- les singletons `espace synthese` identifiés par le code `org` de l'espace de l'organisation.
- les singletons `compte compti compta invits` portant l'identification du compte.
- un document `partition` pour les comptes O.
- tous les documents `avatar note chat sponsoring ticket` dont l'identifiant _majeur_ est celui d'un des avatars du compte.
- tous les documents `groupe note membre` dont l'identifiant _majeur_ est celui d'un des groupes où le compte est _actif_.

Ces documents SAUF `synthese compta partition` sont _synchronisés_: toute modification les concernant est signalée par web-push aux browsers dont une session est connectée et faisant partie de leur périmètre.
- `synthese compta partition` sont a contrario lus sur demande dans les vues qui en affichent le contenu.

**L'objet `adc`:**
- c'est un petit résumé de quelques compteurs de `compta`, qui N'EST PAS synchronisé en raison de la fréquence élevée de ses modifications.
- quand `compta` subit dans le serveur une des modifications _significatives_ ci-dessous, un objet `adc` résumé est retourné en résultat **ET publié par PUBSUB aux autres sessions**:
  - changement de valeur des quotas,
  - changement des _alertes_ détectées sur compta,
  - évolution sensible des compteurs de consommation,
  - changement de la _date limite de validité_ du compte.

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

`./hb-store.js`
- gère le _heartbeat_ avec le service PUBSUB.

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
  - `./services.json` : valeur utilisées en test.
  - `./README.json`
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
- fichier strictement identique en application Web et services OP / PUBSUB.
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
- toutes les classes de _document_ de l'application et leur compilation depuis le format _data_ sérialisé reçu par _Sync_ et lu depuis IDB.

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
- opération `GetEspace` : avec Sync, seule opération utilisée pour gérer connexion / déconnexion / synchronisation.

`util.mjs`
- diverses fonctions utilitaires.

`webcrypto.mjs`
- fonctions utilisant la cryptographie (Web-Crypto).

## Le folder `src/assets`
`./fonts`
- les fontes utilisées dans l'application.

`./help`
- les ressources de l'aide en ligne (voir plus avant).

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
  - _boutons à gauche_: aide, notifications, menu, accueil, page _back_, statut de la session, fichiers visibles en avion, presse-papier
  - _titre de la page courante_. Le cas échéant une seconde barre affiche les onglets pour les pages ayant des onglets.
  - _boutons à droite_: ouverture du drawer de recherche (si la page a une recherche), aide.
- **footer**:
  - _boutons à gauche_: langue, mode clair / foncé, outils,
  - _information du compte connecté_ son type `Délégué / Autonome / Organisation`, son nom, son organisation,
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
- chaque dialogue dans `App` est gardé par un `v-if` de la variable modèle qui l'ouvre.
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
- _ui.paranovis_ : Panel de déverrouillage 

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

Dans ces components le fond N'EST PAS fixé, il est transparent, et suivra celui de l'environnement. Mais celui de la fonte doit l'être, d'où le component suivant:
- `SdNb [texte idx]`: il choisit entre `SdBlanc` et `SdNoir` selon, a) que le mode Quasar _dark_ est actif ou non, b) que idx passé en propriété est absent ou pair ou impair (??? pas clair, à vérifier).

> Remarque: de facto seul `SdNb` est utilisé dans les autres éléments. Et encore car l'affichage d'un MD s'effectue quasiment tout le temps par le component `ShowHtml` qui englobe `SdNb`. Toutefois il existe des cas ponctuels d'utilisation de SdBlanc.

# Opérations
Toutes les opérations sont rassemblées dans `src/app/operations4.mjs`, sauf `Sync (SyncStd / SyncFull)` et `GetEspace` qui sont dans `src/app/synchro.mjs`:
- les opérations standard héritent de `Operation` (src/app/operation.mjs), 
- les opérations `Sync (SyncStd / SyncFull)` héritent de `OperationS` (`src/app/synchro.mjs`) qui hérite de `Operation`.
- une opération a un constructeur acceptant son _code_ en paramètre. Le _code_ est une entrée de traduction dans i18n.
- une opération a une méthode `async run` ayant des paramètres spécifiques de l'opération.
  - elle peut retourner un résultat et se termine par appel de la méthode générique `finOK()`.
  - elle trappe les exceptions et les transmet à la méthode générique `finKO()`.

Exemple:

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
- pour les opérations `Sync`, **toutes** les exceptions sont _tueuses_.

Une exception _tueuse_ affiche la page `clos` qui ne laisse aucune autre solution que de sortir de l'application.

Les exceptions _normales_:
- provoquent l'ouverture du dialogue `DialogueErreur.vue`
- la sortie de ce dialogue a les options suivantes, selon le type d'exception et le choix de l'utilisateur:
  - **se déconnecter**,
  - **se déconnecter et tenter de se reconnecter** au même compte avec la même phrase secrète.,
  - **ne rien faire**, c'est à dire laisser la session se poursuivre _comme si_ l'opération n'avait pas été lancée. Dans ce cas on peut récupérer cette exception dans l'application (APRÈS son passage par `DialogueErreur.vue`).

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
- s'il y a un traitement de `Sync` en cours, la méthode se limite à cette mise en attente.
- sinon la méthode invoque en asynchrone un traitement de `Sync` qui va demander les données correspondantes aux avis de changements reçus. 

### Objet `trlog`
- `sessionId`: `rnd.nc`. Permet de s'assurer que ce n'est pas une notification obsolète d'une connexion antérieure.
- `cid` : **format long seulement** - ID du compte
- `vcpt` : version du compte
- `vesp` : version de l'espace
- `vadq` : version de compta quand adq a _significativement_ changé
- `lag` : liste `[ [idi, vi], ...]` des Couples des IDs des avatars et groupes ayant été impactés avec leur version.
- `lp`: **format long seulement**. `[[compteId, {v, vpe, p}]]` des périmètres des comptes ayant été impactés par l'opération (sauf celui de l'opération initiatrice).
  - `v` : version du compte,
  - `vpe` : version du compte lors de son dernier changement de périmètre,
  - `p` : liste ordonnée des IDs de ses avatars et de ses groupes.

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

Voir dans `src/app/synchro.mjs` les opérations de `Connexion...` et `Sync...`.

## Synchronisation _automatique_, gestion du _heartbeat_
La synchronisation est normalement automatique, les avis de changements des documents par les opérations des **autres** sessions sont reçus par web-push et traités.

La gestion du _heartbeat_ est assurée dans `hb-store` par les actions `doHB stopHB`: un signal de vie est émis vers le service PUBSUB toutes les 2 minutes (10 secondes en cas de _retry_), afin que PUBSUB maintienne en vie le contexte de la session.

MAIS le service PUBSUB peut s'interrompre sans que le service OP ne soit interrompu:
- les opérations peuvent toujours être effectuées MAIS la session ne reçoit plus les avis des autres sessions: son affichage est retardé.
- l'indicateur `hb.statusHB` est à `true` quand le _heartbeat_ a détecté que le service PUBSUB est fonctionnel:
  - il est mis à `true` au lancement du _heartbeat_ et mis à `false` à son arrêt explicite.
  - il est également mis à `false` quand il a été détecté une rupture dans la numérotation des notifications reçues: a priori une interruption du service (arrêt puis relance) a eu lieu.
  - un `pingHB` teste que le service PUBSUB est disponible et `doHB` tente en boucle de ré-initialiser le contexte de la session dans le service PUBSUB.

Quand, soit `hb.statusHB` est `false`, soit `config.permState` est différent de `granted`, la synchronisation automatique **N'EST PLUS ACTIVE**:
- l'utilisateur peut changer son acceptation des notifications si c'était cela qui bloquait.
- il peut déclencher une _synchronisation complète_ explicite si c'était `statusHB` qui bloquait: 
  - si le service PUBSUB est à nouveau _up_, la synchronisation automatique revient à l'état normal.
  - sinon, la synchronisation automatique reste inactive, l'utilisateur ayant à redemander une resynchronisation explicite périodiquement ou en cas de doute sur les données affichées.

`doHB` émet périodiquement une requête POST au service PUBSUB en incrémentant le numéro d'envoi afin de pouvoir détecter le cas échéant une rupture dans la séquence des web-push d'avis de modification des documents du périmètre de la session.

La déconnexion `stopHB` poste aussi un avis au service PUBSUB pour l'informer de la fin de la session et lui permettre de supprimer les données de la session qu'il conserve.

Quand le `statusHB` tombe à `false` de manière inattendue un `doHB` est lancé: en d'autres termes la session tente automatiquement de rétablir le _heartbeat_ (ce qui indirectement provoquera une synchronisation complète): les interruptions _temporaires_ sont normalement surmontées.

L'action `pingHB` teste simplement la disponibilité du service PUBSUB.

> La tombée du service PUBSUB est détectée, soit au prochain _heartbeat_ (au plus dans 2 minutes), soit au retour de la prochaine opération de mise à jour émise par la session.

# L'aide en ligne
Les ressources correspondantes sont toutes dans `/src/assets/help` :
- `_plan.json` : donne la table des matières des pages de l'aide.
- des images en PNG, JPG, SVG comme `dessin.svg`.
- les pages de texte en md : `xxx_lg-LG.md`
  - `xxx` est le _code_ la page.
  - `lg-LG` est la locale (`fr-FR` `en-EN` ...).
  - le _titre_ de la page est une traduction dans `/src/i18n/fr-FR/index.js`
    - son entrée est: `A_xxx: 'Le beau titre',`

### Plan de l'aide
Le plan de l'aide en ligne s'affiche dans les pages d'aide, soit en dessous, soit à droite selon que l'écran est en portrait ou paysage.

Le plan apparaît comme un arbre avec sous chaque rubrique des rubriques annexes.

Le _code_ d'une page d'aide n'est pas lié à sa place dans l'arbre, c'est un code absolu et le plan peut être changé sans modifier les identifiants des pages.

    [
      "DOCpg",  
      ["pages", 
        [ "pages_struct", "top_bar" ],
        ["page_login", "page_login_m", "page_login_pin", ... ],
        ...
      ],
      ["special", "page_admin" ]
    ]

Dans la séquence `["page_login", "page_login_m", "page_login_pin", ... ]`
- `"page_login"` est la page _principale_ de la rubrique,
- `"page_login_m", "page_login_pin", ...` sont les _sous_ rubriques.

### Conventions d'écriture des pages en markdown
**La page est découpée en _sections_**, chacune est identifiée par une ligne:

    # Titre de ma section | page1 page3

Avec un unique espace entre `#` et le texte du titre.

La partie **avant** la première ligne `# section...` est _l'introduction_.

Chaque section est présentée avec:
- une _expansion_ dépliable qui permet d'en voir juste le titre, puis le détail une fois dépliée,
- une liste éventuelle d'autres pages d'aide donnant des précisions sur certains sujets: les codes des pages sont donnés dans la ligne de titre après le signe `|`.

#### Images
Les _images_ apparaissent sous l'une de ces formes:

    SVG dans <img>
    
    <img src="dessin.svg" width="64" height="64" style="background-color:white">
    
    PNG / JPG dans un <img>
    
    <img src="logo.png" width="96" height="96" style="background-color:white">

Ce qui figure dans `src` est le nom du fichier de l'image dans `/src/assets/help`

Le fichier est chargé en tant que ressource en base64 (`src/boot/appconfig.mjs getImgUrl`) et le tag `<img...` est réécrit. En cas d'absence c'est une image par défaut qui est prise.
- na pas oublier le _background_ pour les SVG et PNG.

#### Hyperliens
Un hyperlien est à exprimer par un tag "a":

    <a href="http://localhost:4000/fr/pagex.html" target="_blank">Manuels</a>

    // Ne pas oublier target="_blank" sinon la page va s'ouvrir sur celle de l'application.

Toutefois si ce lien correspond à une page de manuel de la documentation de l'application, on utile la convention suivante:

    <a href="$$/pagex.html" target="_blank">Manuels</a>

Si la ligne commence exactement par `<a href="$$/` Le terme `$$` sera remplacé par l'URL de la documentation de l'application afin d'avoir une aide en ligne indépendante d'une localisation _en dur_.

Le fichier `/public/services.json` a cette forme:

    {
      "docsurls": { 
        "fr-FR": "https://asocialapps.github.io/frdocs", 
        "en-EN": "https://asocialapps.github.io/endocs" 
      },
      ...
    }

Ce fichier est défini au déploiement, après _build_.

# Base locale IDB

Dans le browser il y a une base par compte s'étant connecté en mode synchronisé:
- nom : `'$asocial$-' + '(id du compte) crypté par sa clé K et édité en base 64'`
- ce nom est enregistré en _localStorage_ dans la variable de clé `'$asocial$-' + 'hash du début de la phrase secrète'`. Cette clé est remplacée en cas de changement de phrase secrète.

### Objet _nom de base_ / _trigramme_
- à la création d'une base IDB, l'utilisateur est invité à donner un _trigramme_ (par exemple 'tom').
- en _localStorage_ la variable `$asocial$-trigrammes` donne un texte en base64, qui mis en binaire et désérialisé, donne un objet `trigs` ayant,
  - une propriété par nom de base,
  - dont la valeur est le trigramme associé.

Cet objet permet au propriétaire du browser dans la page de gestion des bases de supprimer les bases qu'il juge obsolètes / encombrantes.

### STORES d'une base IDB
Une base IDB contient les stores suivants:

    const STORES = {
      singletons: 'n', 
      // La clé est le nom du document

      collections: '[id+n+ids]', 
      // La clé est le triplet id, nom ,ids du document

      ficav: 'id', 
      // La clé est l'id du fichier (idf dans une note)

      loctxt: 'id', 
      // La clé est l'id de la note dans le presse-papier

      locfic: 'id', 
      // La clé est l'id du fichier dans le presse-papier

      fdata: 'id' 
      // La clé est l'id du fichier et sa data donne son contenu
    }

#### `singletons`
Le store `singletons` contient les documents singletons pour le compte: `['', 'boot', 'espaces', 'datasync', 'comptes', 'comptis', 'invits']`
- un document est accédé par l'indice `n` de son nom dans la liste ci-dessus.
les deux propriétés sont :
  - `n` : indice du nom.
  - `data` : le document sous forme sérialisée _data_ crypté par la clé K du compte, sauf pour `boot`.

`boot : 1`
- l'item est le triplet `{ n, dh, data }`
  - `n` : 1.
  - `data` est `{ id, clek }` crypté par le PBKFD de la phrase secrète.
    - id du compte,
    - clé K du compte.
  - `dh` : date-heure de dernière écriture.
- `boot` est réécrit en cas de changement de phrase secrète.
- il faut avoir la phrase secrète pour obtenir la clé K du compte ET son id.
- si la date-heure dh est plus ancienne que `IDBOBS` jours, la base est considérée comme perdue et effacée car elle contient un historique trop vieux pour être rafraîchi, aucune connexion ne s'étant opéré depuis `IDBOBS` jours.
- `IDBOBS` est une constante de `api.mjs` (usuellement 18 * 30).

`datasync : 2`
- `data` est l'objet `datasync` sérialisé et crypté par la clé K du compte.
- `datasync` donne l'état courant de IDB, le périmètre du compte avec les versions des documents.

Les autres `singletons` ont pour `data` le document sous forme sérialisée _data_ crypté par la clé K du compte.

#### `collections`
Un item collections a 4 propriétés:
- `id`: id du document crypté par la clé K du compte et mis en base 64.
- `n`: indice du type de documents: `{ avatars: 1, groupes: 2, notes: 3, chats: 4, sponsorings: 5, tickets: 6, membres: 7, chatgrs: 8 }`
- `ids`: ids du document crypté par la clé K du compte et mis en base 64.
- `data`: le document sous forme sérialisée _data_ crypté par la clé K du compte.

#### `ficav`
Chaque item a les propriétés:
- `id`: id du fichier crypté par la clé K du compte et mis en base 64.
- `data`: le document de la classe `Ficav` (dans `src/app/modele.mjs`) sous forme sérialisée _data_ crypté par la clé K du compte.

#### `loctxt`
Chaque item a les propriétés:
- `id`: id du fichier crypté par la clé K du compte et mis en base 64.
- `data`: le document de la classe `NoteLocale` (dans `src/app/modele.mjs`) sous forme sérialisée _data_ crypté par la clé K du compte.

#### `locfic`
Chaque item a les propriétés:
- `id`: id du fichier crypté par la clé K du compte et mis en base 64.
- `data`: le document de la classe `FichierLocal` (dans `src/app/modele.mjs`) sous forme sérialisée _data_ crypté par la clé K du compte.

#### `fdata`
Chaque item a les propriétés:
- `id`: id d'un fichier d'une note, d'un fichier ou d'une note du presse-papier crypté par la clé K du compte et mis en base 64.
- `data`: contenu binaire crypté par la clé K du compte.

### Purge des micro bases locales inutiles
A la première ouverture d'une session synchronisée pour un compte, il est demandé au titulaire du compte des initiales en 3 lettres (ou ce qu'il veut) : ce code est associé au nom effectif de la base locale.

Avant de se connecter à une session, le titulaire d'un compte peut demander à voir la liste des micro bases locales installées dans le navigateur et pour chacun voit:
- les 3 lettres d'initiales données préalablement,
- le nom technique de la micro base locale du compte correspondant.

Il peut pour chacune,
- demander le calcul de son volume utile : le volume technique réel est probablement de près du double, donc savoir si sa suppression libérera ou non un espace significatif.
- en demander la suppression. Elle ne sera plus accessible en mode avion jusqu'à ce qu'une reconnexion en mode synchronisé ne la recharge et elle aura perdu la liste des fichiers accessibles en mode _avion_ (qu'il faudra citer à nouveau).

### Espace privé dans les navigateurs
Chaque navigateur (Firefox, Chrome, etc.) a son propre espace privé pour héberger ces données, **compartimenté** par domaine : si l'application est invoquée par `https://srv1.monhergeur.net/#/monreseau` il y a un espace dédié à `srv1.monhergeur.net`. Pour en voir le contenu,
- passer en mode _debug_ du navigateur (Ctrl-Shift-I pour Chrome et Firefox) et aller sur l'onglet `Application`. La liste des bases y figure.
- on y voit alors les bases listées sur la page de gestion des purges des bases. On peut aussi la supprimer ici ... et en consulter l'état de ses tables totalement abscons car crypté.

Pour faire plus vite, ouvrir la page de gestion des purges des micro bases locales, c'est affiché en clair.

# Annexe: liste des _pages_

### `PageAccueil.vue`
Page affichée après connexion réussie et par appui sur le bouton _Accueil_ de la barre supérieure.
- un premier bloc présente les accès aux pages s'ouvrant aussi depuis les icônes de la barre supérieure de App.
- un second bloc est le menu d'accueil accessible depuis le bouton menu _hamburger_ de App.

Imports:
- `components/MenuAccueil.vue`

### `PageAdmin.vue`
LA page de l'administrateur technique. Elle a 2 onglets:
- **Liste des espaces**: création d'un nouvel espace et un item par espace (organisation) existant.
- **Tâches en cours**: sélection d'un espace particulier ou sinon tâches du GC.
  - un item par tâche,
  - initialisation des tâches du GC pour un hébergement neuf.
  - bouton de lancement du GC.

Dialogues internes:
- `PAcreationesp`: Création d'un espace.
- `PAnvspc`: Changement de la phrase de sponsoring du Comptable.
- `PAedprf`: Changement des quotas de l'espace.

Imports:
- `components/PhraseContact.vue`
- `components/SaisieMois.vue`
- `components/ApercuNotif.vue`
- `components/IconAlerte.vue`
- `components/QuotasVols.vue`
- `components/ChoixQuotas.vue`

### `PageChats`
Affiche la liste des chats des contacts et des groupes.
- si le filtre `filtre.filtre.chats.tous` est `false`, les stores _avatar_ et _groupe_ ne délivrent QUE ceux de l'avatar courant positionné sur la page d'accueil.
- exporte les chats sélectionnés dans un fichier MarkDown.

Imports: 
- `components/MicroChat.vue`
- `components/MicroChatgr.vue`
- `components/ApercuGenx.vue`
- `dialogues/NouveauChat.vue`
- `components/SelAvid.vue`

### `PageClos`
Page ouverte quand la clôture immédiate de la session est la seule issue possible:
- blocage intégral par l'administrateur technique,
- compte résilié par une autre session ou celle courante.

La seule sortie possible de cette page est le retour à la page de login.

### `PageCompta`
Quatre onglets donnent l'état de la comptabilité et des blocages.
- **alertes**: liste des notifications en cours (avec leurs blocages éventuels).
- **Abon./Conso.**: abonnement et consommation (PanelCompta).
- **Crédits**: pour les comptes autonomes seulement (PanelCredits).
- **Chats d'urgence**: chats d'urgence avec le Comptable et les délégués.

Imports:
- `components/PanelCompta.vue`
- `components/ApercuGenx.vue`
- `components/PanelCredits.vue`
- `components/PanelAlertes.vue`

#### Composant `PanelCredits` : auxiliaire de `PageCompta`
- obtention de la statistique des crédits,
- affichage de la liste des crédits,
- affichage de la liste des dons,
- appel de la génération d'un ticket.

Imports:
- `components/ApercuTicket.vue`
- `components/PanelDialtk.vue`
- `components/ApercuGenx.vue`
- `components/SaisieMois.vue`

#### Composant `ApercuTicket` : auxiliaire de `PanelCredits`
- affichage, création, suppression d'un ticket de paiement.

Imports:
- `components/PanelDialtk.vue`

#### Composant `PanelDialtk` : auxiliaire de `PanelCredits` et `ApercuTicket`.
- affichage création mise à jour d'un ticket de paiement.

### `PageCompte`
Affiche les avatars du compte et les opérations du compte:
- création d'un nouvel avatar,
- auto-mutation en compte A,
- mise à jour des quotas du compte,
- suppression d'un avatar.

Dialogues internes:
- `PCmuta`: Dialogue de mutation en compte A.
- `PCnvav`: Dialogue de création d'un nouvel avatar.
- `SAsuppravatar`: Dialogue de suppression d'un avatar.
- `PTedq`: Dialogue de mise à jour des quotas du compte

Imports:
- `components/ApercuAvatar.vue`
- `components/NomAvatar.vue`
- `components/ChoixQuotas.vue`

#### Panel `SupprAvatar` : auxiliaire de `PageCompte`
Panel de suppression d'un avatar ou du compte.
- affiche les conséquences en termes de pertes de notes, de groupes et de chats avec les volumes associés récupérés.

Imports:
- `components/QuotasVols.vue`

#### Composant `ApercuAvatar` : auxiliaire de `PageCompte`
Affiche les données d'identification d'un avatar.

Dialogues internes:
- `AAeditionpc`: Dialogue d'édition de la phrase de contact.

Imports:
- `components/ApercuGenx.vue`
- `components/PhraseContact.vue`

### `PageContactgr`
Page proposant des contacts du compte pouvant être ajoutés à un groupe (n'étant pas déjà membre actif / invité / contact du groupe).

Dialogues internes:
- `PInvit` : Confirmation du contact

Imports:
- `components/ApercuGenx.vue`

### `PageEspace`
Affiche pour le Comptable le découpage de l'espace en partitions:
- création de partition et ajustement de ses quotas,
- acceptation ou non des comptes A,
- quotas réservés aux comptes A,
- nombre de mois d'inactivité.

Dialogues internes:
- `PEedqA`: Dialogue de mise à jour des quotas des comptes A.
- `PEnp`: Dialogue de création d'une nouvelle partition.

Imports:
- `components/SaisieMois.vue`
- `components/ChoixQuotas.vue`
- `components/SynthHdrs.vue`
- `components/SynthLigne.vue`

### `PageFicav`
Affiche la liste des fichiers visible en mode avion et pour chacun,
- permet de l'afficher et de l'enregistrer localement,
- de voir la note à laquelle il est attaché.

Imports:
- `components/MenuFichier.vue`

### `PageGroupe`
Affiche les détails d'un groupe:
- onglet **Détail du groupe**: entête et participations des avatars du compte au groupe.
  - bouton d'ajout d'un contact comme contact du groupe.
- onglet **Membres**: liste des contacts membres du groupe si le compte a accès aux membres.

Dialogues internes:
- `AGediterUna` : Gérer le mode simple / unanime.
- `AGgererheb` : Gérer l'hébergement, changer les quotas.
- `ACGouvrir` : Chat du groupe.

Imports:
- `components/ApercuGenx.vue`
- `components/ApercuMembre.vue`
- `components/SelAvidgr.vue`
- `components/QuotasVols.vue`
- `components/ChoixQuotas.vue`
- `panels/ApercuChatgr.vue`

### `PageGroupes`
Liste les groupes dans lesquels le compte,
- est simple contact,
- est invité en attente d'acceptation / refus,
- est actif.

Affiche:
- la synthèse des volumes occupés par les groupes hébergés,
- un bouton de création d'un nouveau groupe,
- une carte par groupe avec :
  - un bouton pour ouvrir le chat du groupe,
  - un bouton pour accéder à la page du groupe.

Dialogues internes:
- `IAaccinvit` : Acceptation / refus de l'invitation.
- `PGctc` : Création d'un nouveau contact du groupe.
- `PGcrgr` : Création d'un nouveau groupe.
- `PGACGouvrir` : Chat du groupe.

Imports: 
- `components/ChoixQuotas.vue`
- `components/NomAvatar.vue`
- `components/ApercuGenx.vue`
- `components/SelAvid.vue`
- `panels/ApercuChatgr.vue`
- `components/InvitationAcceptation.vue`

### `PageLogin`
Login pour un compte déjà enregistré ou auto-création d'un compte depuis une phrase de sponsoring déclarée par un sponsor.

Dialogues internes:
- `ASaccsp`: Dialogue d'acceptation d'un nouveau sponsoring.
- `pubsub`: Dialogue de demande de permission de notification.

Imports: 
- `components/PhraseContact`
- `panels/AcceptationSponsoring`

#### Panel `AcceptationSponsoring` : auxiliaire de `PageLogin`
Saisie de l'acceptation d'un sponsoring, in fine création du compte (si acceptation).
- saisie du nom,
- saisie de la phrase secrète,
- saisie du mot de remerciement.

Imports:
- `components/EditeurMd.vue`
- `components/ShowHtml.vue`
- `components/QuotasVols.vue`

### `PageNotes`
Affiche l'arbre des notes avec pour racines les avatars et les groupes:
- en tête affiche le détail de la note courante, avec les actions qu'elle peut subir.
- la barre séparatrice permet de lancer le chargment local des notes sléctionnées et le plier / déplier global de l'arbre.
- en bas l'arbre des notes selon leur rattachemnt.

Dialogues internes:
- `NE` : Edition du texte de la note.
- `NX` : Gestion de l'exclusivité de la note.
- `NF` : Gestion des fichiers attachés à la note.
- `AP` : Album de photo de la note et ses descendantes.
- `confirmSuppr` : Confirmation de la suppression d'une note.
- `PNdl` : Dialogue de download des notes sélectionnées.
- `NM` : Mise à jour des hashtags de la note.

Imports:
- `components/ShowHtml.vue`
- `panels/NoteEdit.vue`
- `panels/NoteExclu.vue`
- `panels/NoteFichier.vue`
- `components/ListeAuts.vue`
- `components/NotePlus.vue`
- `components/HashTags.vue`
- `components/BoutonConfirm.vue`
- `components/ApercuGenx.vue`
- `panels/AlbumPhotos.vue`

#### Panel `NoteEdit` : auxilaire de `PageNotes`
Editeur du texte d'une note après choix de son auteur pour une note de groupe.

Imports:
- `components/EditeurMd.vue`
- `components/ListeAuts.vue`
- `components/NoteEcritepar2.vue`
- `components/ApercuGenx.vue`
- `components/NodeParent.vue`

#### Panel `NoteExclu` : auxilaire de `PageNotes`
Gestion de l'exclusivité d'écriture d'une note de groupe.

Imports:
- `components/ApercuGenx.vue`
- `components/ListeAuts.vue`
- `components/NodeParent.vue`

#### Panel `NoteFichier` : auxilaire de `PageNotes`
Gestion des fichiers attachés à une note.

Dialogues internes:
- `NFouvrir` : Dialogue de création d'un nouveau fichier.
- `Zimg` : Zoom d'une photo

imports:
- `dialogues/NouveauFichier.vue`
- `components/MenuFichier.vue`
- `components/NoteEcritepar2.vue`
- `components/ZoomPhoto.vue`
- `components/ListeAuts.vue`
- `components/NodeParent.vue`

#### Dialogue `NouveauFichier` : auxiliaire de `NoteFichier`
Dialogue par étapes pilotant la création d'un nouveau fichier ou nouvelle révision d'un fichier, gestion des fichiers à supprimer à cette occasion. In fine suivi des étapes d'enregistrement sur le _stockage_ et le serveur.

Imports:
- `components/NomGenerique.vue`

#### Panel `AlbumPhotos` : auxilaire de `PageNotes` et `NoteFicchier`
Présente toutes les _miniatures_ des fichiers _images_ attachés aux notes sous une racine connée.

Dialogues internes:
- `Zimg` : Zoom d'une photo

Imports:
- `components/ZoomPhoto.vue`

### `PagePartition`
Pour un délégué et le Comptable, affiche les données de la partition et la la liste de ses comptes.
- pour le Comptable, celle désignée depuis la page de l'espace.
- pour un délégué, SA partition.

Dialogues internes:
- `NSnvsp`: Dialogue de création d'un nouveau sponsoring.
- `PTedq`: Dialogue de mise à jour des quotas du compte.

Imports:
- `components/SynthHdrs.vue`
- `components/SynthLigne.vue`
- `components/ApercuNotif.vue`
- `components/ChoixQuotas.vue`
- `components/ApercuGenx.vue`
- `components/QuotasVols.vue`
- `panels/NouveauSponsoring.vue`
- `components/BarrePeople.vue`

#### Composant `SynthHdrs` : auxiliaire de `PageEspace` et `PagePartition`
Aaffichage des entêtes de colonnes des partitions.

#### Composant `SynthLigne` : auxiliaire de `PageEspace` et `PagePartition`
Affichage de la ligne d'une partition.

Dialogues internes:
- `PEedqP` : Dialogue de mise à jour des quotas de la partition.
- `PEedcom` : Édition du code d'une partition.
- `DNdialoguenotif` : Dialogue de gestion d'alerte niveau partition.

Imports:
- `components/ChoixQuotas.vue`
- `components/DialogueNotif.vue`

### `PagePeople`
Affiche tous les contacts connus avec une courte fiche pour chacun.
- un bouton rafraîchit les cartes de visite qui en ont besoin.

Imports:
- `components/ApercuGenx.vue`

### `PageSession`
Page qui s'affiche pendant l'initialisation de la session, après login et avant la page d'accueil. Elle s'affiche ausii sur demande par le bouton _Résumé de la session en cours_ de la page d'accueil.
- **État général** de la session et de sa consommation cumulée depuis le début de la session.
- **RapportSynchro**: son contenu est dynamique lors du chargement de la session, puis fixe après (synthèse du chargement initial).

Imports:
- `components/RapportSynchro.vue`

#### Composant `RapportSynchro` : auxiliaire de de `PageSession`
- affiche par avatars et groupes du compte, leurs nombres de notes, membres, sponsorings, chats, tickets.

### `PageSponsorings`
Liste les sponsorings actuellement en cours ou récents:
- boutons de prolongation des sponsorings en cours et d'annulation.

Dialogues internes:
- `NSnvsp` : Dialogue de création d'un sponsoring: pour un délégué et un compte "O" dans sa propre partition.

Imports:
- `components/ShowHtml.vue`
- `panels/NouveauSponsoring.vue`
- `components/QuotasVols.vue'`

# Annexe: liste des _panels_

### Panel `ApercuChat` : ouvert par `ChatsAvec` `MicroChat`
Ce panel affiche une entête d'information sur le chat et affiche dans l'ordre antéchronologique les items d'échanges. Des dialogues d'actions s'applique, soit au niveau du chat, soit au niveau d'un item.

Dialogues internes:
- `mutation` : Gestion des mutations.
- `BPmut` : Mutation de type de compte en "type" A ou O.
- `ACconfirmeff` : Confirmation d'effacement d'un échange.
- `ACconfirmrac` : Confirmation du raccrocher.
- `BPcptdial` : Affichage des compteurs de compta du compte "courant".
- `ACchatedit` : Dialogue d'ajout d'un item au chat.

Imports:
- `components/SdBlanc.vue`
- `components/EditeurMd.vue`
- `components/ApercuGenx.vue`
- `components/PanelCompta.vue`
- `components/ChoixQuotas.vue`

### Panel `ApercuChatgr` : ouvert par `PageGroupes` `PageGroupes` `MicroChatgr`
Ce panel affiche une entête d'information sur le chat du groupe et affiche dans l'ordre antéchronologique les items d'échanges. Des dialogues d'actions s'applique, soit au niveau du chat, soit au niveau d'un item.

Dialogues internes:
- `ACGconfirmeff` : Confirmation d'effacement d'un échange.
- `ACGchatedit` : Dialogue d'ajout d'un item au chat.

Imports:
- `components/SdBlanc.vue`
- `components/EditeurMd.vue`
- `components/SelAvmbr.vue`
- `components/ApercuGenx.vue`

### Panel `DialogueHelp` : ouvert par `BoutonHelp`
Affiche les pages d'aide.
- la page d'aide courante.
- l'arbre des pages d'aide.

Cet arbre est construit depuis `src/assets/help/_plan.json`
- chaque page ppp est un fichier `src/assets/help/ppp_fr-FR.md`, voire en d'autres langues.

Les titres des rubriques d'aide (des pages d'aide) sont traduits dans `src/i18n`.

Dialogues internes:
- `readme` : Affichage du README de la version courante de l'application.

Imports:
- `components/ShowHtml.vue`

### Panel `NouveauSponsoring` : ouvert par `PageSponsorings` `PagePartition`
Panel de saisie par étapes d'un sponsoring par un compte lui-même sponsor.

Imports:
- `components/NomAvatar.vue`
- `components/ChoixQuotas.vue`
- `components/EditeurMd.vue`
- `components/PhraseContact.vue`
- `components/QuotasVols.vue`

### Panel `OutilsTests` : ouvert par `App` et `PageAccueil`
Plusieurs onglets:
- **Comptes synchronisés:**
  - présente la liste des bases synchronisées dans le browser.
  - sur demande calcul de leur volume.
  - propose la suppression de la base.
- **Tests d'accès:** 
  - tests d'accès au serveur, 
  - ping des bases locales et distantes,
  - simulation d'erreur.
- **Affichage du thème:** couleurs et fontes.
- **Test d'une phrase secrète:** affichage de ses cryptage / hash.
- **_Test du calcul des compteurs:_**
  - pas visible en production.
  - permet de tester le recalcul des compteurs selon une succession d'étapes dans le temps.

Dialogues internes:
- `OTrunning` :  Dialogue de calcul du volume d'une base locale.
- `ORsupprbase` : Dialogue de suppression d'une base locale.

Imports:
- `components/TestCompteurs.vue`

### Panel `PressePapier` : ouvert par un bouton de `App`
Gère le presse-papier des notes et fichiers conservés localement en mémoire et/ou dans la base locale.

Dialogues internes:
- `PPnvnote` : Dialogue de confirmation de suppression d'une note.
- `PPnvfic` : Dialogue d'ajout d'un nouveau fichier.
- `PPsupprnote` : Dialogue de confirmation de suppression d'une note.
- `PPsupprfic` : Dialogue de confirmation de suppression d'un fichier.

Imports:
- `components/ShowHtml.vue`
- `components/EditeurMd.vue`
- `components/NomGenerique.vue`








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

### Dialogue `ApercuCv` : ouvert par `ApercuGenx` `ListeAuts`
Affiche la photo et le texte d'une carte de visite.

Imports:
- `components/ShowHtml.vue`

### Dialogue `CarteVisite` : ouvert par `ApercuGenx`
Edition d'une carte de visite, photo et texte.
- la photo peut être issue d'un fichier ou de la caméra.
- elle peut être retaillée et centrée.

Imports:
- `webcam-easy`
- `vue-advanced-cropper`
- `components/EditeurMd.vue`

### Dialogue `ChoixEmoji` : ouvert par `EditeurMd`
Dialogue de saisie des émojis à insérer dans un input / textarea.
- se ferme à la fin de la saisie.
- singleton, du fait de son inclusion dans EditeurMd qui n'a qu'une seule instance en édition à un instant donné (toujours inclus dans un dialogue).

Import:
- `emoji-mart-vue-fast/data/google.json`
- `emoji-mart-vue-fast/src`

### Dialogue `DialogueErreur` : ouvert par `App`
Affiche une exception `AppExc` et gère les options de sortie selon sa nature (déconnexion, continuation ...).

Bien qu'affiché par `App`, la sollicitation vient de `ui-store.afficherExc`:
- sur trap d'une exception de `Operation`,
- sur trap d'exceptions UI:
  - `util.trapex` : `MenuFichier NouveauFichier`
  - `PageNotes`

### Dialogue `Parano` : ouvert par `App`
Affiche plein écran un clavier simplifié permettant de saisir le code PIN de déverrouillage (donc de disparition du dialogue) ou de retour à la page de login.
- déclenché périodiquement ou appui sur l'icône de verrou.

### Dialogues `DialStd1`
Template générique d'un dialogue simplifié avec,
- une barre de titre,
- un slot,
- une barre de boutons renocer / valider.

### Dialogues `DialStd2`
Template générique d'un dialogue pleine page (avec _layout_) avec,
- une barre de titre,
- un slot dans le _page-container_.

### Dialogue `NouveauChat` : ouvert par `ChatsAvec` `MicroChat` `PageChats`
Dialogue de création d'un _chat_ avec un premier item.

Imports:
- `components/PhraseContact.vue`
- `components/ApercuGenx.vue`
- `components/EditeurMd.vue`

### Dialogue `PhraseSecrete` : ouver par `App`
Saisie contrôlée d'une phrase secrète et d'une organisation (sur option), avec ou sans vérification par double frappe.

Bien que techniquement ouvert par `App`, de facto PhraseSecrete l'est depuis:
- `PageLogin`: saisie de la pharse de connexion.
- `AcceptationSponsoring`: donnée de la phrase par le filleul juste avant sa connexion.
- `PageCompte`: changement de phrase secrète.
- `PageCompta`: saisie de la phrase secrète du Comptable à la création d'un espace.
- `OutilsTests`: pour tester la saisie d'une phrase secrète et la récupération de ses cryptages.

Ce composant héberge *simple-keyboard* qui affiche et gère un clavier virtuel pour la saisie de la phrase. Il utilise pour s'afficher un `<div>` de classe `simple-keyboard` ce qui pose problème en cas d'instantiation en plusieurs exemplaires.
- Ceci a conduit a avoir une seule instance du dialogue hénergée dans `App` et commandée par la variable sorres.`ui.d.PSouvrir`
- les propritées d'instantiation sont dans `stores.ui.ps`, dont `ok` qui est la fonction de callback à la validation de la saisie.
- le dialogue est positionné au *top* afin de laisser la place au clavier virtuel de s'afficher au dessous quand il est sollicité.

Imports:
- `simple-keyboard`
- `simple-keyboard/build/css/index.css`

# Annexe: _components_ particuliers

### Filtres
La page principale `App` a un drawer à droite réservé à afficher les filtres de sélection propres à chaque page et permettant de restreindre la liste des éléments à afficher dans la page (par exemple les notes).

Chaque filtre est un component simple de saisie d'une seule donnée: la valeur filtrée étant stockée en `filtre-store`.

- **FiltreAinvits**: case à cocher pour filtre des groupes ayant une invitation en cours.
- **FiltreAmbno**: filtre des membres d'un groupe selon leurs drots d'accès:
  - '(indifférent)',
  - 'aux membres seulement',
  - 'aux notes seulement',
  - 'aux membres et aux notes',
  - 'ni aux membres ni aux notes',
  - 'aux notes en écriture'
- **FiltreAvecgr**: case à cocher 'Membre d\'un groupe' pour filtre des contacts.
- **FiltreAvecmut**: case à cocher pour filtre des chats ayant une demande de mutation en cours.
- **FiltreAvecsp**: case à cocher pour filtrer les comptes délégués d'une partition.
- **FiltreAvgr**: case à cocher pour filtrer les notes de groupes.
- **FiltreEnexcedent**: case à cocher 'Groupes en excédent de volume' dans la liste des groupes.
- **FiltreInvitables**: case à cocher pour filtrer les seuls contacts pouvant être invités dans un groupe.
- **FiltreMc**: liste de _hashtags_ (qui peuvent être soit requis, soit interdits) pour toutes les pages ayant un filtre (sauf `admin` et `partition`).
- **FiltreNbj**: saisie d'un nombre jours 1, 7, 30, 90, 9999 pour les pages `chats` et `notes`.
- **FiltreNom**: saisie d'un texte filtrant le début d'un nom ou un texte contenu dans un string (presque toutes les pages).
- **FiltreNonlus**: case à cocher pour filtrer les _chats_ non lus.
- **FiltreNotif**: filtre pour sélectionner les comptes d'une `partition` en fonction de la gravité de sa _notification_.
- **FiltreRac**: menu de sélection d'un statut de chat:  
  - '(tous, actifs et raccrochés)',
  - 'Chats actifs seulement',
  - 'Chats raccrochés seulement'
- **FiltreSansheb**: case à cocher pour filtrer les 'Groupes sans hébergement'.
- **FiltreStmb**: menu de sélection du statut majeur d'un membre.
  - '(n\'importe lequel)',
  - 'contact',
  - 'invité',
  - 'actif',
  - 'animateur',
  - 'DISPARU',
- **FiltreVols**: menu permettant de sélectionner un volume de fichiers d'une note 1Mo, 19Mo, 100,Mo 1Go

#### `FiltreTri`
Ce filtre spécifique sélectionne un critère de tri dans une des listes `TRIadmin TRIespace TRIpartition` définies au dictionnaire.

### Boutons

Ils n'importent aucune autre vue et sont des "span" destinés à figurer au milieu de textes.
- **BoutonBulle**: affiche en bulle sur clic, un texte MD figurant dans le dictionnaire des traductions.
- **BoutonBulle2**: affiche en bulle sur clic, un texte MD qui a été composé dynamiquement en respectant les traductions.
- **BoutonConfirm**: active la foncion de confirmation quand l'utilisateur a frappé le code aléatoire de 1 à 255 qui lui est proposé.
- **BoutonDlvat**: voir détail plus avant.
- **BoutonHelp**: ouvre une page d'aide.
- **BoutonLangue**: affiche la langue courante et permet de la changer.
- **BoutonUndo**: affiche une icône _undo_, disable ou non selon la condition passée en propriété.

#### Boutons spéciaux / icônes
- **IconAlerte**: affiche un statut d'alerte / restriction.
- **IconMode**: affiche le mode de connexion courante.
- **QueueIcon**: petit rond de couleur au-dessus d'une icöne pour marquer l'existence d'une queue de fichiers en téléchargement.

#### Bouton `BoutonDlvat` : inséré dans `PageAdmin`
Saisie d'un mois signifiant une date limite de connexion autorisée par l'Administrateur Technique.

Dialogues internes:
- `PEdlvat` : Changement d'une _dlvat_.

# Annexe: autres _components_

(TODO - A réviser)

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

