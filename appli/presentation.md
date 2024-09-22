---
layout: page
title: Présentation Générale
---

Une organisation, association, un groupe de quelques amis ... peut décider de disposer de son propre **espace a-social**, par exemple `monasso`. Il lui faut:
- soit trouver un _prestataire_ qui l'accepte et dont les conditions générales de ventes, prix, éthique, etc. lui conviennent.
- soit décider d'être son propre _prestataire_ et de déployer l'application dont le logiciel est disponible _open source_: ceci nécessite quelques compétences techniques et a minima une heure d'effort.

In fine le _prestataire_ a initialisé un espace accessible par une URL comme `https://s1.monhebergeur.net` 

Pour se connecter à son espace, il suffit d'ouvrir dans un navigateur la page Web à cette adresse et de fournir: 
- `monasso` : le code de son organisation enregistrée par l'hébergeur, 
- `mabellephrasetressecrete` une phrase secrète personnelle d'au moins 24 signes.

> Derrière cette URL, un _serveur_ délivre l'application Web et gère les accès à une base centrale où les données de chaque organisation sont stockées. Jusqu'à 60 organisations étanches les unes des autres peuvent être gérées depuis une même URL, l'enregistrement d'une nouvelle organisation prenant moins d'une minute.

> Une organisation n'est pas attachée à son prestataire initial qui peut _exporter_ les données de l'organisation pour un autre, ce qui ne prend que le temps de la copie des données.

## Les comptes et leurs avatars

Avant de pouvoir accéder à l'application une personne doit créer son propre compte **sponsorisé par un autre compte**. Sponsor et sponsorisé ont convenu entre eux,
- du nom du sponsorisé, par exemple `Charles`, qui pourra changer.
- d'une preuve de sponsoring par une expression comme `le hibou n'est vraiement pas chouette`.

Le _sponosirisé_ initie la création de son compte après avoir cité le nom de l'organisation `monasso` et la phrase preuve de sponsoring.

Si le titulaire du nouveau compte accepte les conditions proposées par le sponsor, il finalise la création de son compte en déclarant sa **propre phrase secrète de connexion**.
- elle a au moins 24 signes et reste uniquement dans la tête du titulaire, n'est enregistrée sous aucune forme nulle-part et pourra être changée à condition de pouvoir fournir celle actuelle.
- le début de la phrase ne doit pas être le même que celui d'une phrase déjà enregistrée afin d'éviter de tomber _par hasard_ sur la phrase d'un autre compte.

> La phrase secrète crypte indirectement toutes les données du compte aussi bien dans la base centrale que dans les micro bases locales de chacun des navigateurs utilisés par le compte. Un piratage des appareils des titulaires des comptes ou de la base centrale ne donnerait au _pirate_ que des informations indéchiffrables.

> _Revers de cette sécurité_ : si la personne titulaire d'un compte oublie sa **phrase secrète de connexion**, elle est ramenée à l'impuissance du pirate. Son compte s'autodétruira dans un délai d'un an et toutes ses données et notes disparaîtront.

### Avatars principal et secondaires facultatifs d'un compte

En créant son compte, le titulaire a créé son **avatar principal**. 

Un avatar,
- a un _identifiant_ aléatoire de 12 lettres / chiffres: ce code est immuable et non interprétable.
- a une **carte de visite** cryptée constituée,
  - d'une photo facultative,
  - d'un court texte (son _nom / pseudo_) d'au moins 6 signes, par exemple `Charles III, roi des esturgeons et d’Écosse`.

Ultérieurement le titulaire du compte peut créer des **avatars secondaires**, 
- chacun a son identifiant et sa carte de visite. 
- le titulaire du compte peut utiliser à son gré l'un ou l'autre de ses avatars, ce qui lui confère plusieurs _personnalités_ vis à vis des autres.

> **Le titulaire du compte est le seul à connaître ses avatars secondaires**. En regardant deux avatars, personne n'est en mesure de savoir s'ils correspondent ou non au même compte.

> Comme dans la vraie vie **plusieurs avatars peuvent porter le même "nom"**: à l'écran les 4 derniers signes de l'identifiant complète les noms ( `Charles#9476` et `Charles#5AR2`). Leurs cartes de visite permettront aux autres de distinguer Charles "_Général de brigade_" de Charles "_Roi des esturgeons_".

### L'administrateur technique
C'est le représentant technique du prestataire. **Il n'a pas de compte** mais _une clé d'accès à l'application_ pour initialiser un espace pour une organisation et effectuer quelques actions techniques: exportation d'espaces, suppressions d'espaces, notifications importantes, etc.

### Le Comptable
_Le Comptable_ **est un compte** qui a reçu de l'administrateur technique à la création de l'espace de l'organisation une _phrase de sponsoring_ qui lui a permis de créer son compte. C'est un compte _presque_ normal, toutefois: 
- mais pour être bien identifiable il a une _carte de visite_ immuable, sans photo et de nom _Comptable_,
- il ne peut pas se créer des avatars secondaires,
- il a le pouvoir de gérer **des forfaits gratuits**:
  - en découpant le forfait global **en tranches**,
  - en pouvant désigner certains comptes comme _délégués_ et en charge d'accorder des forfaits gratuits à des comptes.

> Le **Comptable** n'a pas plus que les autres comptes les moyens cryptographiques de s'immiscer dans les notes des avatars des comptes et leurs chats: ce n'est en aucune façon un modérateur et il n'a aucun moyen d'accéder aux contenus, pas plus qu'à l'identité des avatars, sauf de ceux qu'il connaît personnellement.

### Les comptes autonomes "A"
Un compte _autonome_:
- **paye son abonnement** (qu'il fixe lui-même) **et sa consommation** (sans limite a priori),
- **ne peut être ni _restreint_, ni _bloqué_** tant que son solde est créditeur.

Un compte "A" _augmente son solde_ en faisant parvenir des _paiements_ que le Comptable va enregistrer sans qu'il puisse faire le rapprochement entre un _paiement_ et le compte crédité. 

### Les comptes de l'organisation "O"
Pour certaines organisations, les comptes "A" ne sont pas acceptables:
- si un compte "A" quitte l'organisation ou qu'il est devenu nuisible à l'organisation, il ne peut pas être restreint / bloqué.
- l'organisation peut souhaiter contrôler les ressources utilisées par les comptes et les restreindre,
- l'organisation peut avoir inclus l'abonnement et la consommation de certains comptes dans ses _frais d'adhésion_ ou équivalents.

Pour répondre à ces objectifs, il existe une seconde catégorie de compte: **les comptes "O", de l'organisation**.

L'organisation paye de facto l'abonnement et la consommation pour le compte mais en contrepartie,
- elle lui fixe des limites potentiellement bloquantes d'abonnement et de consommation,
- elle peut restreindre voire bloquer le compte. 

[Information détaillée à propos de la gestion des comptes](./comptes.html)

[Créer, gérer et supprimer ses avatars](./avatars.html)

## Notes personnelles

**Une note porte un texte** d'au plus 5000 caractères pouvant s'afficher avec un minimum de _décoration_, gras, italique, titres, listes ... Ce texte est modifiable.

**Des fichiers peuvent être attachés à une note**
- les types de fichiers les plus usuels (`.jpg .mp3 .mp4 .pdf ...`) s'affichent directement dans le navigateur, les autres sont téléchargeables.
- il est possible d'ajouter et de supprimer des fichiers attachés à une note.
- quand plusieurs fichiers portant le même _nom_ dans la note, ils sont vus comme des _révisions_ successives, qu'on peut garder, ou ne garder que la dernière, ou celles de son choix.

### Vue hiérarchique: note _parent_ d'un note
- les notes apparaissent à l'écran sous forme hiérarchique, une note _parent_ ayant en dessous d'elle des notes _enfants_ (ou aucune).
- les notes n'ayant pas de note _parent_ apparaissent rattachées à celui des avatars du compte a qui elle appartient: cet avatar est une _racine_ de la hiérarchie des notes.

> Un avatar peut créer des notes **personnelles**, les mettre à jour, les supprimer et **les indexer par des mots clés personnels**. Elles sont cryptées, comme toutes les données des comptes, et seul le titulaire du compte a, par l'intermédiaire de sa phrase secrète, la clé de cryptage apte à les décrypter.

[Notes personnelles](./notes.html)

## Contacts

Un contact est _un avatar_ dont le compte connaît **la carte de visite** cryptée. 

### Contact _permanent_
Un contact _permanent_ est établi entre deux avatars quand ils ont ouvert un _chat_ entre eux (un _chat_ ne pouvant pas être supprimé). La clé de la carte de visite a été échangée à la création du _chat_.

### Contact _temporaire_
Lorsque **deux avatars sont membres d'un même groupe**, les clés de leur cartes de visites sont inscrites dans le groupe et sont ainsi accessibles à tous les membres (ayant droit d'accès aux membres). 

**Ce contact est temporaire**, dure tant que les deux avatars sont membres du groupe.
- si l'un des deux avatars souhaite rendre ce contact **permanent** il ouvre un _chat_ avec l'autre.
- un tel contact _disparaît_ quand l'avatar correspondant est résilié du groupe ou disparaît.

> Un compte peut rester totalement isolé et n'avoir _aucun contact_ avec les autres: à la création de son compte par _sponsoring_, le sponsor comme le sponsorisé peuvent déclarer vouloir ou non **ouvrir un _chat_** entre eux (ce qui les rend _contacts mutuels permanent_).

> Un contact est réciproque, si Julie a Émilie dans ses contacts, Émilie a Julie dans ses contacts: chacun a échangé avec l'autre la clé de cryptage qui permet de lire la carte de visite de l'autre.

### Commentaire personnel et _hashtags_ attachés à ses _contacts_ 
Ils ont pour but de faciliter le filtrage dans le répertoire de ses _contacts_. Le commentaire et les hashtags attachés à un contact, sont spécifiques du compte, lui seul peut les décrypter. La _carte de visite_ d'un contact pouvant évoluer selon la volonté de ce dernier, conserver un nom / commentaire personnel à son propos est une bonne idée.

> Les _hashtags_ attribués par un compte à un contact lui permettent de le classer comme _expert_ _oubliette_ ou _ami_ et de s'en servir comme filtre du répertoire de ses contacts et récupérer par exemple tous les _expert_ sauf les _amis_ ...

## "Chats" entre avatars

Un chat peut être **ouvert**,
- à la création du compte entre sponsor et sponsorisé si tous deux en sont d'accord,
- avec le membre _actif_ d'un groupe pour lequel on a droit d'accès aux membres (voir plus loin),
- par contact direct avec une _phrase de contact_.

Les deux avatars peuvent écrire des textes courts:
- un texte d'un _chat_ ne peut plus y être modifié mais peut être supprimé par son auteur,
- le volume total des textes sur le _chat_ est limité à 5000 signes, les plus anciens étant perdus en cas de dépassement de cette limite.

Une fois créé un _chat_ ne disparaît que quand les deux avatars qui le partage ont disparu.
- pour ne pas être importuné, l'un des 2 peut _déclarer le chat indésirable_, ce qui en efface le contenu pour lui. Le _chat_ n'est plus décompte plus pour lui dans son nombre de documents. L'autre peut toujours continuer à y écrire des textes sans être certain d'être lu ... Le _chat_ n'est plus _indésirable_ dès que l'avatar qui l'a déclaré tel y écrit un texte (et compte à nouveau dans son décompte de documents).
- chacun peut attacher au _contact du chat_ ses propres _hashtags_ (par exemple _copain_ ou _important_ ...) que l'autre ne voit pas, et les utiliser pour filtrer ses _chats_.

### Contact par une _phrase de contact_ d'un avatar
Émilie peut déclarer une _phrase de contact_, unique dans l'application et dont le début n'est pas déjà le début d'une phrase déjà déclarée. Par exemple : `les courgettes sont bleues au printemps`
- Émilie peut la changer ou la supprimer à tout instant.
- Émilie peut communiquer, par un moyen de son choix, cette phrase à Julie qui peut ainsi ouvrir un _chat_ avec elle.
- Julie et Émilie ayant un _chat_ ouvert, sont devenues contacts _permanents_. L'une pourra aussi inviter, ou faire inviter, l'autre aux groupes auxquels elles participent.

> Une _phrase de contact_ doit être effacée rapidement afin d'éviter que des personnes non souhaitées, mises au courant de la phrase de contact, n'ouvrent un _chat_: l'impact serait toutefois limité (on n'est pas obligé de le lire).

[Plus d'information sur les contacts et les "chats"](./contactschats.html)

## Les groupes et leurs notes

Un avatar peut créer un **groupe** dont il sera le premier membre _actif_ et y aura un pouvoir _d'animateur_. Un groupe a,
- un identifiant interne et **une carte de visite** (comme un avatar).
- un **chat partagé par les membres du groupe**, les membres sont des _avatars_.
- un **des notes partagées entre les membres** qui peuvent les lire et les éditer.

Un avatar connu dans un groupe peut avoir plusieurs états successifs:
- **simple contact**: il a été inscrit comme contact du groupe mais lui-même ne le sait pas et ne connaît pas le groupe.
- **contact invité**: un membre actif ayant pouvoir d'animateur a invité le contact à devenir membre actif. L'avatar invité voit cette invitation et s'il l'accepte deviendra membre actif, sinon il retournera à l'état de simple contact. Nul ne devient membre actif à son insu.
- **membre actif**: il peut participer à la vie du groupe.

### Accès aux membres et / ou aux notes
Un membre actif _peut_ recevoir lors de son invitation deux _droits_:
- **droit d'accès aux autres membres** et au _chat_ (ou non),
- **droit d'accès aux notes** en lecture, en lecture et écriture ou pas du tout.

Lors de son invitation il peut aussi recevoir le **pouvoir d'animation**. S'il ne l'a pas, un membre _animateur_ peut lui conférer ce pouvoir (mais ne pourra plus lui enlever).

> **Certains groupes peuvent être créés à la seule fin d'être un répertoire de contacts** cooptés par affinité avec possibilités de _chat_. Personne n'y lit / écrit de notes.

> **Certains groupes peuvent être créés afin de partager des notes _anonymes_ de discussion**: par exemple un animateur est seul à avoir droit d'accès aux membres, à les connaître: les notes sont de facto anonymes pour les autres membres.

En général les groupes sont créés avec le double objectif de réunir des avatars qui se connaissent mutuellement, échangent sur le chat et partagent des notes.

### Commentaire personnel et _hashtags_ attachés à un groupe
Tout membre actif peut attacher un commentaire personnel et ses propres _hashtags_ à un groupe. Personne d'autre n'en a connaissance, son commentaire et ses _hashtags_ restent strictement privés.

La recherche d'un groupe quand on est membre de beaucoup de groupes en est facilitée.

### Notes d'un groupe
- elles sont cryptées par la clé aléatoire spécifique au groupe qui a été transmise à chaque membre lors de son invitation au groupe.
- hormis les membres actifs du groupe ayant droit d'accès aux notes, personne ne peut accéder aux notes du groupe.
- quand un nouveau membre accepte une invitation au groupe avec droits d'accès aux notes, il a immédiatement accès à toutes les notes existantes du groupe. S'il redevient _simple contact_ ou perd son droit d'accès aux notes (de par sa volonté ou celle d'un _animateur_), il n'a plus accès à aucune de celles-ci (ce qui allège ses sessions).
- pour écrire / modifier / supprimer une note du groupe, il faut avoir le droit d'accès en écriture aux notes.
- chaque note contient la succession des membres qui y sont intervenu.

### _Hashtags_ d'un membre à une note d'un groupe
Ceci facilite les recherches d'un compte dans ses notes.
- Le filtrage par _hashtags_ s'effectue en session tous groupes confondus. 
- Les autres membres ne savant pas quels sont ces _hashtags_.
- Un _animateur_ peut attacher des _hashtags publics_ à une note: ils sont visibles de tous les membres.

### Vue hiérarchique: note de groupe _parent_ d'une autre note du même groupe
Ceci fait apparaître visuellement à l'écran une hiérarchie.

**Une note personnelle peut avoir pour _parent_ une note de groupe** pour la compléter / commenter: toutefois l'avatar propriétaire de la note personnelle sera seul à la voir (puisqu'elle est _personnelle_).

### Membre _hébergeur_ d'un groupe
_L'hébergeur du groupe_ est un membre qui s'est dévoué pour supporter les coûts d'abonnement de stockage (nombres de notes et volume des fichiers) des notes du groupe.

[Information détaillée à propos des groupes et de leurs notes](./notes.html)

## Modes de connexion *synchronisé*, *incognito* et *avion*

Pour se connecter à son compte, le titulaire d'un compte choisit sous quel **mode** sa session va s'exécuter: _synchronisé_, _avion_ ou _incognito_.

### Mode "normal" _synchronisé_ 
C'est le mode préférentiel où toutes les données du périmètre d'un compte sont stockées dans une micro base locale cryptée dans le navigateur remise à niveau depuis le serveur central à la connexion d'une nouvelle session puis maintenue à jour en cours de session.

Une connexion ultérieure après une session synchronisée est rapide et économique: l'essentiel des données étant déjà dans le navigateur, seules les _mises à jour_ sont tirées du serveur central.

### Mode _avion_
Pour que ce mode fonctionne il faut qu'une session antérieure en mode _synchronisé_ ait été exécutée dans ce navigateur pour le compte. A la connexion le titulaire du compte y voit l'état dans lequel étaient ses données à la fin de sa dernière session synchronisée dans ce navigateur. **L'application ne fonctionne qu'en lecture**.

> On peut couper le réseau (le mode _avion_ sur un mobile), de façon à ce que l'ouverture de la page de l'application ne cherche même pas à vérifier si une version plus récente est disponible.

### Mode _incognito_
**Aucun stockage local n'est utilisé, toutes les données viennent du serveur central**, l'initialisation de la session est plus longue qu'en mode synchronisé. Aucune trace n'est laissée sur l'appareil (utile au cyber-café ou sur le mobile d'un.e ami.e).

> On peut ouvrir l'application dans une _fenêtre privée_ du navigateur, ainsi même le texte de la page de l'application sera effacé en fermant la fenêtre.

> **En utilisant des sessions synchronisées sur plusieurs appareils, on a autant de copies synchronisées de ses notes et chats sur chacun de ceux-ci**, et chacun peut être utilisé en mode avion.

[En savoir plus sur les modes, l'accessibilité des fichiers en mode _avion_, le _presse-papier_](./modessync.html)

## Coûts d'hébergement de l'application

Le coût d'usage de l'application pour une organisation correspond aux coûts d'hébergement des données et de traitement de celles-ci. Selon les techniques et les prestataires choisis, les coûts unitaires varient mais existent dans tous les cas.

### _Base de données_ et _fichiers_ (Storage)
Leur stockage ont des coûts unitaires très différents (variant d'un facteur de 1 à 6).
- les _bases de données_ requièrent un stockage proche du serveur et des accès très rapides,
- les fichiers sont enregistrés dans des _Storage_, des stockages techniques distants ayant une gestion spécifique et économique du fait d'être soumis à peu d'accès (mais de plus fort volume).

### Abonnement: coût de l'espace occupé en permanence
Un abonnement correspond aux coûts récurrents mensuels pour un compte:  même quand il ne se connecte pas, le stockage de ses sonnées a un coût.

L'abonnement est décomposé en deux lignes de coûts correspondant à l'occupation d'espace en base de données et en _storage_:
- **Prix unitaire de stockage d'un document** multiplié par le **nombre total de _documents_**: notes personnelles et notes d'un groupe hébergé par le compte, chats personnels non _indésirables_, nombre de participations actives aux groupes.
- **Prix unitaire du stockage des fichiers dans un _storage_** multiplié par le **volume total des fichiers attachés aux notes**.

Pour obtenir le coût correspondant à ces deux volumes il est pris en compte, non pas _le volume effectivement utilisé à chaque instant_ mais forfaitairement **les _volumes maximaux_ forfaitaires** auquel le compte est abonné (ses _quotas_).

> Les volumes _effectivement utilisés_ ne peuvent pas dépasser les volumes maximum de l'abonnement. Dans le cas où un changement de l'abonnement réduit a posteriori ces maximum en dessous des volumes utilisés, les volumes n'auront plus le droit de croître.

### Consommation : coût de calcul et de transfert des fichiers
La consommation correspond à l'usage effectif de l'application quand une session d'un compte est ouverte. Elle comporte 4 lignes:
- **nombre _de lectures_** (en base de données).
- **nombre _d'écritures_** (en base de données).
- **volume _descendant_** (download) de fichiers téléchargés en session depuis le _storage_.
- **volume _montant_** (upload) de fichiers envoyés dans le _storage_ pour chaque création / mise à jour d'un fichier.

### Coût total mensuel
Il correspond au total de l'abonnement (2 lignes) et de la consommation (4 lignes) valorisées par un **tarif** fixé par le prestataire:
- un tarif est simplement la données des 6 coûts unitaires des 6 éléments d'abonnement (2) et de consommation (4).

>_L'ordre de grandeur_ du prix de revient total pour un compte varie en gros de **0,5€ à 3€ par an**. Individuellement ça paraît faible mais n'est plus du tout négligeable pour une organisation assurant les frais d'hébergement d'un millier de comptes ...

### Gestion des _abonnements gratuits_ des comptes "O"
Le Comptable procède d'abord à un _découpage en partitions_ des ressources globales dont il dispose:
- chaque _partition_ a un quota de _nombre de documents_, de _volume de fichiers_ et de _consommation de calcul_.
- tout compte "O" est attaché à une _partition_.
- pour chaque _partition_ le Comptable peut (ou non) confier une _délégation_ à certains comptes de la tranche afin que ceux-ci,
  - fixent pour chaque compte "O" de leur partition des quotas d'abonnement et de consommation,
  - puissent gérer des _notifications_ à ces comptes (avec restriction éventuelle).

[En savoir plus sur les partitions des comptes "O"](./partitions.html)

## Notifications et restrictions d'accès des comptes

Une _notification_ est un message suffisamment important pour que sa présence soit signalée par une icône dans la barre d'entête de l'écran de l'application.

Une _notification_ **peut être porteuse d'une restriction d'accès**: quand une session a une ou des restrictions d'accès, ses actions sont plus ou moins limitées.

### Notification de _l'administrateur technique_ à l'espace de l'organisation
**Elle peut n'être une simple information** (_arrêt programmé ..._) sans restriction mais peut aussi fixer l'une de ces deux restrictions:
- **Espace figé**. Aucune écriture ne peut être faite, typiquement afin de procéder à une opération technique d'export, verrouillage d'une archive d'un espace ... mais peut aussi être une mesure de rétorsion.
- **Espace clos**. L'administrateur technique a effacé les données de l'espace: il ne subsiste plus que cette notification dont le texte donne la raison et le cas échéant indique si l'espace est accessible à une autre adresse.

### Notification du _Comptable_ ou de ses _délégués_ pour les comptes "O"
Le Comptable ou ses délégués peuvent inscrire une _notification_:
- adressée à tous les comptes d'un _partition_,
- adressée à un compte "O" spécifique.

Une notification **peut ne porter qu'un message d'information**, sans restriction.

Elle **peut aussi porter une des deux restrictions**: _lecture seule_ et _accès minimal_.

#### Lecture seule
En lecture seule une session **ne peut que consulter les données** (comme en mode _avion_).
- les échanges sont toutefois possibles sur les _chats d'urgence_ avec le Comptable et les _délégués de sa partition_.

#### Accès minimal
En accès minimal une session **ne peut plus ni lire ni mettre à jour** ses données.
- les échanges sont toutefois possibles sur les _chats d'urgence_ avec le Comptable et pour un compte "O" les _délégués de sa partition_.

**Les connexions du compte ne le maintiennent plus en vie**: au plus tard dans un an, si cette restriction n'est pas levée, le compte disparaîtra.

Une telle restriction pour un compte "O" est causée par:
- déclaration explicite du Comptable ou un de ses délégués.
- automatiquement quand la consommation mensuelle moyenne dépasse la limite fixée.

#### Volume en réduction
Cette restriction bloque toutes les actions menant à une augmentation de volume:
- _création_ d'une note, d'un chat, ou acceptation d'une invitation à un groupe,
- _remplacement_ d'un fichier attaché à une note par un autre plus important.

Cette restriction est **automatique**, causée par le dépassement des limites d'abonnements (nombre de documents, volume des fichiers).

### Restriction _accès minimal_ pour excès de consommation d'un compte "A"
En accès minimal une session ne peut plus ni lire ni mettre à jour ses données, toutefois:
- les échanges sont possibles sur les _chats d'urgence_ avec le Comptable.
- les opérations de crédit / gestion des volumes maximaux restent autorisées.

Elle est **automatique** lors de la détection d'un solde négatif.

> **En cas de _restriction_ les connexions des comptes ne les maintiennent plus en vie**: au plus tard dans un an, si cette restriction n'est pas levée, les comptes disparaîtront.

[Les notifications et restrictions en détail](./notifications.html).

## Statistiques partagées entre le Comptable et l'administrateur technique

Tous deux ont besoin d'éléments statistiques de consommation afin d'ajuster si nécessaire les bases de facturation en fonction d'un usage réel.

Ces statistiques sont calculées mensuellement et sont des fichiers CSV.
- elles sont anonymes: les identifiants des comptes concernés n'y figurent pas.
- pour un mois donné elles sont immuables, calculées à un moment où le mois étant terminé, les compteurs mensuels ne sont plus susceptibles de changer.

**Abonnement / consommation des comptes**

**Archive des _tickets_ de paiement des comptes "A"**

[Détail des statistiques](./stats.html)

## Gestion des _espaces_

**L'administrateur technique** d'un site peut y héberger techniquement jusqu'à 60 **espaces**.

Tout ce qui précède se rapporte à UN espace et les utilisateurs n'ont aucune perception des autres espaces hébergés par le même serveur technique.
- dans la base de données, les informations sont partitionnées.
- dans l'espace de stockage des fichiers, des sous-espaces sont séparés par nom de l'organisation.

> Certaines organisations peuvent souhaiter avoir plus d'un espace pour elle: par exemple un espace de _production_, un espace de _démonstration / training_, un espace _archive récente_ ...

L'administrateur technique a la possibilité d'ouvrir _instantanément_ un nouvel espace pour une organisation en faisant la demande. 
- Le Comptable et l'administrateur technique se sont mis d'accord sur le volume utilisable et la participation aux frais d'hébergement.
- Cette ouverture crée une phrase de _sponsoring_ à destination du Comptable de l'organisation, 
- Le Comptable créé son compte en utilisant cette phrase de _sponsoring_ et en fixant sa phrase secrète.

_Rappel_; l'administrateur technique peut,
- émettre une notification d'information visible de tous les comptes,
- bloquer l'espace de l'organisation en _lecture seule_,
- détruire les données par clôture de l'espace ne laissant pendant un certain temps qu'une seule information d'explication.
