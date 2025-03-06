---
layout: page
title: Analyse de la confiance dans l'application en exécution
---

> Cette note comporte un mix _d'éléments humains_ et de _considérations techniques_. Selon la _culture_ de chacun, elle peut apparaître comme une collection d'évidences ou de discours abscons.

## Confiance humaine
Au delà de la pétition de principe que in fine tout aboutit à une confiance humaine, le plus important est d'analyser ce qui se passe si elle est prise en défaut.

On peut consacrer des moyens considérables à essayer de casser une clé de cryptage, mais le fait est que la corruption et la menace sur son propriétaire sont en général des procédés moins coûteux pour obtenir le même résultat.

### Une structure non hiérarchique qui _confine_ les conséquences des fuites des _utilisateurs_
**Dans les structures _dictatoriales_, un _sommet_ détient plus d'information que les niveaux inférieurs**. Mais la structure de l'application **n'est pas** hiérarchique mais à plat, en réseau:
- _faire tomber_ la confidentialité d'un compte ne fait tomber qu'une partie de la confidentialité:
  - celles des notes du compte,
  - celles des _groupes_ dont le compte est membre, notes et / ou liste des membres. Si les notes révèlent un contenu, la liste des membres ne révèlent que des cartes de visite où le propriétaire a mis ce qu'il voulait (et rarement son adresse postale et son identité réelle).
  - celles des chats avec d'autres comptes, dont là encore le contenu de l'échange est connu et la carte de visite peut-être non fiable.
- il semble très difficile d'imposer aux comptes de toujours inclure un _responsable hiérarchique_ dans leurs groupes, sauf par une menace forte au cas où un compte récalcitrant se ferait prendre à ne pas le faire -ce qui reste aléatoire-.

En conséquence il n'y pas de moyens pour faire tomber en masse les comptes d'un espace en _tapant sur une tête_: la perte de confidentialité est toujours restreinte, ce qui **confine** les risques humains d'infiltration / corruption / menace.

### Les _agents_ techniques directs et de second niveau
Plusieurs _agents techniques_ interviennent à différents niveaux, les conséquences de leurs défaillances sont analysés dans la suite de ce document:
- **le développeur** : il a écrit le logiciel, est responsable du contenu de son _source_ initial, pas des transformations et extensions qui peuvent se produire ensuite.
- **l'agent de déploiement de l'application Web** : depuis le source de l'application Web il a configuré les adresses des services centraux à accéder et a _déployé_ sur un site Web l'application pour qu'elle puisse être invoquée par les utilisateurs.
- **l'administrateur technique** a effectué le travail équivalent pour déployer sur un _serveur_ (pour simplifier) l'application _serveur_ (les deux services des _opérations_ et de _push web_).

Les _agents de second niveau_ sont, en vue simplifiée, ceux des _providers_ de service externes:
- _provider du site Web_ d'où l'application Web peut être invoquée.
- _provider du serveur central_, ou des _cloud functions_ correspondantes.
- _provider de la base de données_ responsable d'assurer la confidentialité des données qui y sont stockées.
- _provider du storage_ ou les fichiers attachés aux notes sont stockés.

Ce qui est analysé ci-dessous n'est pas la _fiabilité / disponibilité_ de leurs prestations mais leur capacité à assurer la _confidentialité_ vis à vis des autres bien entendu mais aussi d'eux-mêmes. Plus prosaïquement, le risque zéro n'existant pas,
- comment déceler une éventuelle faille de confidentialité ?
- que se passe-t-il, quelles conséquences, une faille (toujours possible) de confidentialité a-t-elle ?

## Confiance technique de base
C'est celle que nous sommes obligés d'avoir dans certains éléments techniques, faute de quoi rien n'est possible. Quelques uns sont listés ci-dessus.

### Le couple _matériel / Operating System + browser_ des terminaux
Un terminal peut avoir des _portes dérobées_ qui par exemple,
- trace les frappes au clavier,
- liste des données en mémoire de l'application en cours d'exécution, comme un debugger à distance non contrôlé par le propriétaire du terminal.

Sans confiance dans ces matériels / logiciels aucune confidentialité n'est possible à assurer: les textes à afficher sont _en clair_ dans la mémoire des postes sinon ... ils ne peuvent pas s'afficher, les clés de cryptage y sont aussi en clair sinon ... elles ne peuvent ni crypter ni décrypter.

> L'application Web permet de saisir sa phrase secrète sur un _clavier_ applicatif: ceci empêche d'intercepter les frappes au clavier depuis un _clavier hacké_. Si c'est la mémoire du _browser_ qui est accessible depuis l'extérieur, certes la saisie n'est pas interceptée mais son résultat l'est.

> De la paranoïa ? Peut-être mais des mobiles piratés à distance ont fait l'actualité et la rumeur dit que les géants du Web ont leur propre version de Linux dans leurs serveurs pour éviter d'être hackés à leur insu.

### Le couple _matériel / Operating System_ des serveurs
Quand le serveur est hébergé par un _provider_ externe, on en revient à la question plus générale d'avoir confiance en lui ou non.

Quand on est son propre hébergeur, la confidentialité est compromise si le _provider de la VM_ (pour simplifier) a placé des _portes dérobées_ permettant de lire les données en cours d'exécution dans les processus exécutant Node.

## Conséquences des compromissions selon les _niveaux / agents techniques_
### _Données humaines_ versus _meta-données_
**Les _données humaines_** sont les textes et fichiers humainement interprétables,
- les textes des notes et de leurs fichiers attachés, des chats, des cartes de visite et leurs photos,
- par extension les clés de cryptage permettant de les crypter / décrypter.

Les _identifiants_ des comptes, avatars, notes, chats, groupes ... sont des chaînes de 12 chiffres / lettres latines minuscules et majuscules aléatoires. Un identifiant ne _signifie_ rien tout seul. 

**Les _meta-données_ sont des _relations_ entre identifiants**:
- liste des avatars d'un compte,
- liste des groupes accédés par un avatar / compte,
- liste des invitations aux groupes d'un avatar / compte,
- liste des comptes sponsorisés par un compte A en attente de réponse,
- liste des _contacts_ d'un avatar / compte,
- liste des tickets de crédits d'un compte (leur numéros, pas leur contenu).

Par extension sont des _meta-données_,
- les compteurs d'abonnement / consommation d'un compte,
- la date de fin de validité d'un compte,
- les _numéros de version_, le nombre de fois où un document (compte, avatar, note ...) a été modifié.

### Règles majeures
**Aucune _donnée humaine_ ne sort en clair de la mémoire d'une session de l'application Web** en cours d'exécution sur un _terminal_.
- même ce qui s'écrit sur la _micro base de données locale_ est crypté par la clé majeure du compte.

Autrement dit à la double condition,
- que le terminal soit _techniquement digne de confiance_,
- que le logiciel de l'application Web qui s'y exécute n'ait pas subi de transformation depuis son source,

il est impossible de lire aucune _données humaines_ sur aucun autre support technique que la mémoire d'une session.

**Les _meta-données_ ne sont lisibles en clair QUE dans la mémoire des serveurs en cours d’exécution**:
- les sessions en exécution dans un browser ont aussi en mémoire en clair les meta-données mais seulement celles relatives au périmètre du compte auxquels elles sont connectés.

**La base données** n'expose pas les _meta-données_, elles sont cryptées par la même clé qui permet à un serveur de les interpréter dans sa mémoire.

### Conséquences sur le détournement de la base de données
- Les _meta-données_ n'y sont pas lisibles en clair: il faut en faire un extrait et les décrypter par la clé de l'administrateur technique.
- Les _données humaines_ sont inaccessibles, cryptées par les données des comptes.

### Conséquences sur le détournement de l'espace de _storage_ des fichiers
Il n'y a pas de _meta-donnée_ stockées. Les contenus des fichiers sont cryptés et leurs _path_ sont cryptés depuis les identifiants (organisation, ID d'avatar / compte, ID d'un fichier). 

# Problématique de confiance
On suppose que le _matériel / Operating System_ est de confiance.

On suppose aussi que les algorithmes de hash et de cryptage sont fiables:
- SHA-256 et PBKFD.
- AES-256.
- RSA-2048.

Aucune information _publique_ ne permet aujourd'hui d'affirmer le contraire, du moins jusqu'à, peut-être, une diffusion massive de calculateurs quantiques.

### Logiciels _open source_
Tous les logiciels utilisés par l'application sont _open source_, leurs sources peuvent être lus et analysés librement.

Ceci s'arrête aux frontières suivantes:
- certains browsers sur les terminaux: _Chrome_ n'est pas open source.
- l'exécutable de Node sur les serveurs.
- certains logiciels de base de données.

> **Ce n'est pas parce que le logiciel de l'application est _open source_ que pour autant c'est celui-là qui s'exécute !**

A partir de ces prémisses la _confiance_ globale repose sur les points suivants:
- **l'application Web: celle en exécution est-elle celle originale ?**
- **l'application Serveur: celle en exécution est-elle celle originale ?**
- **la clé de cryptage de l'administrateur technique sur les serveurs est-elle restée secrète ?**

> Si les réponses à ces 3 questions sont OUI, la confiance dans l'application est acquise: les réponses dépendent de chaque choix de déploiement. 

# Dissociation entre l'application Web et les serveurs

Un _serveur_ (pour simplifier, ce peut être des _cloud functions_) est joignable à partir de 3 données:
- une URL pour invoquer les services OP (opérations),
- une URL pour invoquer le service PUBSUB (_web push_ des avis de modifications aux browsers),
- une clé publique VAPID nécessaire pour faire fonctionner le _web push_.

Ces trois données,
- sont spécifiques de chaque _serveur_,
- ne sont pas confidentielles. Au pire un utilisateur malicieux tentera des connexions qui échoueront. Mais tout serveur est déjà toujours la cible potentielle d'une attaque par déni de service (saturation).

**N'importe qui connaissant le dépôt github de l'application Web et ces 3 données pour un serveur S1, peut déployer sur un simple site Web statique l'application Web accédant à S1.**

Par exemple `github.com pages`,
- offre une capacité d'hébergement gratuite et publique de sites Web statiques,
- l'habileté technique pour y déployer l'application Web est de faible niveau (pas de développement, mais savoir utiliser les lignes de commandes).

> Une organisation peut parfaitement décider de procéder elle-même à ce déploiement sur ce qui sera _son_ site d'où l'application Web peut être appelée par ses membres depuis un browser.

Voir en annexe la discussion sur la restriction éventuelle d'origine qu'un serveur _peut_ mettre en place.

> A la limite, un membre du groupe très suspicieux vis à vis de son organisation, peut tout à fait avoir _son site_, rien que pour lui ou quelques ami.e.s, de distribution de l'application Web.

Dans les deux cas il appartient à ceux qui ont déployé l'application Web à titre _personnel_ ou de _l'organisation_ de penser à la redéployer en cas d'évolution de ses sources.

> Normalement un prestataire proposant un ou des serveurs, met aussi à disposition un site Web statique permettant à ses organisations clientes d'y accéder.

# MAIS l'application Web _distribuée_ est-elle celle originale ?

Depuis les sources d'origine, **un agent de déploiement de l'application Web**, peut,
- **soit utiliser la configuration par défaut**: dans ce cas l'application _distribuable_ est disponible directement sur le github.
- **soit vouloir ajuster certaines constantes de configuration** dans le fichier de configuration. Il doit alors:
  - obtenir l'application Web sur son poste de travail par un _clone_ standard depuis le dépôt github.
  - éditer localement sur son poste la configuration, voire avec prudence certaines traductions.
  - enregistrer ces _patchs_ dans le site de distribution afin que ceux qui le souhaitent puissent juger de leur non nocivité.
  - procéder localement à un _build_, une _pseudo compilation_ générant le code distribuable (ce qui prend environ une minute).

Une fois la _distribution_ préparée, qu'elle ait été récupérée telle quelle ou après une _configuration + build_ locale, il suffit de modifier dans le _distribuable_ le fichier qui contient les trois données d'accès au serveur de son choix.

## S'assurer en exécution que l'application Web distribuée sur le site n'a pas été _patchée_
### Cas avec _configuration par défaut_
La _distribution_ ne comporte que quelques fichiers, dont en fait 3 seulement sont sensibles. Ces fichiers peuvent être téléchargées depuis l'URL de distribution, sans même avoir un compte dans l'application ni à ouvrir l'application.
- il suffit de vérifier qu'ils sont identiques à ceux du dépôt git _source_ (outil standard de l'OS disponible sous Linux et Windows).

### Cas avec _configuration modifiée_
Le site de distribution doit placer dans le répertoire de distribution un répertoire _patch_ contenant les sources patchés.
- il faut localement refaire un _build_ en prenant les sources initiaux et les patchs.
- il faut ensuite vérifier que les 3 fichiers distribuables sensibles sont conformes.

## Conclusion
Il est simple de disposer de son propre site de _distribution_ pour son organisation.

Il est aussi simple pour n'importe qui de s'assurer que ce site n'a pas été patché malicieusement.

**Une structure indépendante de _certification_** peut opérer et déclarer certifiées les nouvelles versions distribuées (ou prêtes à l'être).

Tout repose ensuite sur la confiance dans les agents ayant généré la distribution et en ceux l'ayant certifier (éventuellement soi-même). Toutefois la facilité de détection d'un comportement malicieux et le nombre de personnes capables de la vérifier réduit fortement le risque de piratage.

Dans ce contexte toutes les _données humainement interprétables_ sont **de confiance**.

# L'application Serveur: celle en exécution est-elle celle originale ?

**L'administrateur technique** a quelques opérations à exécuter pour déployer l'application _Serveur_ depuis les sources _certifiées_ disponibles sur git:
- ajuster la configuration dans le fichier _config_,
- ajouter les _données secrètes_ qui ne sont pas dans git: tokens d'authentification vis à vis des providers, et quelques autres dont **la clé de cryptage des données dans la base**.
- effectuer un _build_ ou non selon les directives de son provider du service,
- déployer le résultat chez son provider _Serveur_.

**Le résultat de cette séquence est invérifiable:**
- une équipe externe de _certification_ ne peut pas obtenir le résultat déployé chez le _provider serveur_,
- elle ne peut donc pas s'assurer que le code initial n'a pas été patché à des fins répréhensibles.

**Il est impossible de s'assurer en exécution que le code qui s'exécute sur le serveur est bien celui qui a été déployé:** il faut faire confiance au provider du serveur. En effet techniquement un service qui s'exécute ne renvoie pas sur demande,
- ni la signature du container Node qui s'exécute,
- ni la signature du code déployé.

#### Les _providers_ ayant pignon sur rue sont-ils de confiance ?
Clairement Google, Microsoft, Amazon Web Services ne s'amusent pas, ni à déformer le code distribué, ni à ouvrir des portes dérobées permettant de patcher / lire / tracer la mémoire des process en exécution.
- techniquement ils peuvent le faire,
- sur demande _insistante_ d'une autorité de sécurité nationale, le feraient-ils ?

### Alternative: assurer soi-même le rôle de provider _serveur_
C'est toujours possible et on peut toujours avoir son propre cloud privé où les VMs sont _safe_.

La question de confiance est ramenée à celle dans l'équipe d'administration ... constituée d'humains corruptibles et pouvant subir des pressions, mais où un protocole de vérification humain est possible.

# La clé de cryptage de l'administrateur technique sur les serveurs est-elle restée secrète ?

Cette clé permet à un intervenant extérieur ayant accès en lecture à la base de données, d'en lire les données dans les champs cryptés (colonnes `_data_`) qui contient les _meta-données_.

Le _provider_ d'accès à la base de données est-il en mesure de lire tous les champs de la base ?
- bien entendu, sinon il ne pourrait pas gérer la base,
- mais les colonnes / attributs `_data_ id ids hk ...` ne lui sont accessibles qu'en binaire crypté dont il n'en a pas la clé. Il ne peut rien en faire et donc ne peut pas techniquement rendre visible les _meta-données_.

> Remarque: l'administrateur technique gère la clé de cryptage et le jeton d'accès à la base. S'il détourne l'une il peut aussi détourner l'autre.

**L'administrateur technique** est en mesure technique de diffuser ces clés à un _hacker_ qui sera en mesure d'accéder à la base et d'extraire / diffuser les _meta-données_.

# Conclusion
**Les _données humaines_ ne sont accessibles que dans la mémoire des sessions en exécution de l'application Web.**
- c'est celle qui _peut_ être le plus facilement certifiée par une équipe autonome de celle l'ayant déployée.
- cette équipe _peut_ s'assurer assez rapidement que le code déployé n'est pas malicieux, ni ne dirige vers des serveurs malicieux.

**Il existe des moyens techniques et organisationnels pour s'assurer que les _données humaines_ sont effectivement protégées.**

**Mais seule la confiance dans une chaîne organisationnelle permet d'estimer que le code qui s'exécute sur le serveur n'est pas malicieux**. Cette même chaîne doit aussi s'assurer de la confidentialité du fichier donnant la clé de cryptage en base de donnes.

**Si cette chaîne est jugée efficace, les _meta-données_ sont aussi protégées.**

Si ce n'est pas le cas, au pire les _meta-données_ peuvent fuiter: on n'en tire pas grand-chose, seulement des relations entre identifiants aléatoires anonymes qui ne peuvent être vraiment exploités que s'ils sont rapprochés des personnes physiques et morales dans le vrai monde.

# Annexe I: contrôler les _origin_ dans un _serveur_

Le _serveur_ peut être configuré pour n'accepter en entrée que des requêtes émises par **des origines** listées explicitement.

Un browser émet dans les requêtes sortantes un _header origin_ comportant le nom de domaine et le port du site Web d'où il a chargé la page émettrice de la requête.

Un _serveur_ peut spécifier quelles _origin_ il accepte (ou toutes par défaut). C'est un moyen pour éviter que des sites Web _parasites_ ne viennent soumettre des requêtes au _serveur_ (par simplification l'autre _header_ user-agent est passé sous silence).

**Mais, ce n'est qu'une sécurité factice** et qui ne contraint que ceux qui utilisent un site Web et un browser pour gérer l'application _cliente_.

Le contrôle de l'origine repose sur le fait que les browsers observent cette obligation, ce qui est le cas de ceux ayant pignon sur rue.

**MAIS n'importe qui peut facilement écrire une application cliente**, dans le langage de son choix, s'exécutant sur un poste, voire une application _mobile_, (mais pas dans un browser) qui écrira dans le header _origin_ de ses requêtes ce qu'elle veut, typiquement l'une de celles attendues par le _serveur_.

> Le serveur N'A AUCUN MOYEN de savoir si le header _origin_ qu'il reçoit est bien une vraie origine écrite par un vrai browser _fair_ ou celle produite par une application pirate qui s'est camouflée en browser.

En ce sens, l'agent de déploiement d'un serveur qui restreindrait les origines, typiquement à celles de son site délivrant l'application Web, n'apporte absolument aucune garantie de sécurité réelle: en revanche il bloque celles de ses organisations clientes ne lui faisant pas confiance aveugle pour l'application Web, ce qui d'une certaine façon n'est pas un signe incitant à la confiance.

# Annexe II: peut-on _authentifier_ un logiciel ?

Un _matériel_, un ordinateur, est identifiable: sa carte d'accès réseau a un identifiant absolu unique dans le monde par fabrication. Cet identifiant est accessible à tout logiciel s'exécutant sur le matériel en question.
- toutefois il faut un logiciel pour transmettre cet identifiant à un serveur, et ce logiciel fait ce qu'il veut, en particulier transmettre un faux identifiant.

Un _humain_ peut avoir des identifiants, _login mots de passe / phrase secrète_ qui ont la caractéristique d'être _externes_ aux ordinateurs qu'il utilise: sa _mémoire cérébrale_ n'y est pas connectée (du moins aujourd'hui).
- il faut bien un logiciel pour transmettre cette _phrase secrète_:
  - d'abord ce n'est pas la phrase qui est transmise mais un hash (en fait deux) de cette phrase.
  - ensuite le serveur peut savoir si oui ou non ce hash correspond à un compte chez lui, il peut croire avoir _authentifié_ la personne ayant émis cette phrase (sans avoir à la connaître), mais en fait il n'en a authentifié qu'un _hash_.

**Remarque**: dans le cas de AsocialApp, si le _serveur_ s'est fait berné en recevant un _hash_ piraté, il renverra à l'application Web des données cryptés par la _vraie phrase secrète entière_ (pas par son hash). Si l'application cliente pirate n'avait que les hash, elle ne peut rien en faire. La _vraie_ phrase secrète en clair ne sort JAMAIS de la mémoire d'une session Web, il faut avoir hacké le browser pour l'exposer (ça arrive mais ce n'est pas très public et il faut savoir s'en servir et la _faille_ ne vit pas forcément longtemps).

**Un logiciel n'a pas d'identification inviolable.**
- **son identifiant est forcément inscrit dans son code**, il ne peut aller le chercher nulle part à l'extérieur.
- s'il est dans le code, avec plus ou moins d'effort, on peut le retrouver. Tous les logiciels sont in fine _décompilables_, les langages de script plus facilement que les autres. Même quand l'identifiant est crypté dans le code, la clé de décryptage y figure aussi ...

Tout cela pour dire qu'un _serveur_,
- **PEUT authentifier un HUMAIN à distance** parce que l'identifiant en question est _externe_ au matériel et au _logiciel_,
- **NE PEUT PAS AUTHENTIFIER un logiciel A** qui communique avec lui parce que son identifiant étant inscrit dans le logiciel "A" lui-même peut être décompilé et réutilisé par un autre logiciel "B" qui pourra ainsi de faire passer pour "A" vis à vis du serveur.

L'application Web, ne peut pas, être _authentifiée_ avec sûreté du fait même de la nature intrinsèque d'un logiciel qui est un texte lisible (humainement **et** par un CPU).
