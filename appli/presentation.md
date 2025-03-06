---
layout: page
title: Présentation Générale
---

Une organisation, association, un groupe de quelques amis ... peut décider de disposer de son propre **espace a-social**, par exemple `monasso`. Il lui faut:
- soit trouver un _prestataire_ qui l'accepte et dont les conditions générales de ventes, prix, éthique, etc. lui conviennent.
- soit décider d'être son propre _prestataire_ et de déployer l'application dont le logiciel est disponible _open source_: ceci nécessite quelques compétences techniques et a minima une heure d'effort.

In fine le _prestataire_ a initialisé un espace accessible par une URL comme `https://asocialapps.github.io/s1?monasso` 

Pour se connecter à son compte, son titulaire ouvre dans un navigateur la page Web à cette adresse et saisit: 
- `monasso` : le code de son organisation, mais il est pré-initialisée dans l'URL après `?`, 
- `mabellephrasetressecrete` sa phrase secrète personnelle de connexion d'au moins 24 signes.

> Derrière cette URL, un _serveur_ délivre l'application Web et gère les accès à une base centrale où les données de chaque organisation sont stockées. Plusieurs organisations étanches les unes des autres peuvent être gérées depuis une même URL, l'enregistrement d'une nouvelle organisation prenant moins d'une minute.

> Une organisation n'est PAS attachée à son prestataire initial qui peut _exporter_ les données de l'organisation pour un autre prestataire, ce qui ne prend que le temps de la copie des données.

## Les comptes et leurs avatars

Avant de pouvoir accéder à l'application une personne doit créer son propre compte **sponsorisé par un autre compte**. Sponsor et sponsorisé ont convenu entre eux,
- du nom du sponsorisé, par exemple `Charles`, qui pourra changer.
- d'une preuve de sponsoring par une expression comme `le hibou n'est vraiment pas chouette`.

Le _sponosirisé_ initie la création de son compte après avoir cité le nom de l'organisation `monasso` et la phrase preuve de sponsoring.

Si le titulaire du nouveau compte accepte les conditions proposées par le sponsor, il finalise la création de son compte en déclarant sa **propre phrase secrète de connexion**.
- elle a au moins 24 signes et reste uniquement dans la tête du titulaire, n'est enregistrée sous aucune forme nulle-part et pourra être changée par son titulaire à condition de pouvoir fournir celle actuelle.
- le début de la phrase ne doit pas être le même que celui d'une phrase déjà enregistrée afin d'éviter de tomber _par hasard_ sur la phrase d'un autre compte.

> La phrase secrète crypte indirectement toutes les données du compte aussi bien dans la base centrale que dans les micro bases locales de chacun des navigateurs utilisés par le compte. Un piratage des appareils des titulaires des comptes ou de la base centrale ne donnerait au _pirate_ que des informations indéchiffrables.

> _Revers de cette sécurité_ : si la personne titulaire d'un compte oublie sa **phrase secrète de connexion**, elle est ramenée à l'impuissance du pirate. Son compte s'autodétruira dans un délai d'un an et toutes ses données et notes disparaîtront.

### Avatars principal et secondaires facultatifs d'un compte

En créant son compte, le titulaire a créé son **avatar principal**. 

Un avatar,
- a un _identifiant_ aléatoire de 12 lettres / chiffres: ce code est immuable et non interprétable.
- a une **carte de visite** cryptée constituée,
  - d'une photo facultative,
  - d'un court texte (son _nom / pseudo_) d'au moins 6 signes, par exemple `Charles III, roi des esturgeons et d’Écosse`. Ce texte peut en dire beaucoup plus, mais sous la seule responsabilité du titulaire.

Ultérieurement le titulaire du compte peut créer des **avatars secondaires**, 
- chacun a son identifiant et sa carte de visite. 
- le titulaire du compte peut utiliser à son gré l'un ou l'autre de ses avatars, ce qui lui confère plusieurs _personnalités_ vis à vis des autres.

> **Le titulaire du compte est le seul à connaître ses avatars secondaires**. En regardant deux avatars, personne n'est en mesure de savoir s'ils correspondent ou non au même compte.

> Comme dans la vraie vie **plusieurs avatars peuvent porter le même "nom"**: à l'écran les 4 derniers signes de l'identifiant complète les noms ( `Charles#9476` et `Charles#5AR2`). Leurs cartes de visite permettront aux autres de distinguer Charles "_Général de brigade_" de Charles "_Roi des esturgeons_".

### L'administrateur technique
C'est le représentant technique du prestataire. **Il n'a pas de compte** mais _une clé d'accès à l'application_ pour initialiser un espace pour une organisation et effectuer quelques actions techniques: exportation d'espaces, suppressions d'espaces, notifications importantes, etc.

### Le Comptable
_Le Comptable_ **est un compte** qui a reçu de l'administrateur technique à la création de l'espace de l'organisation une _phrase de sponsoring_ qui lui a permis de créer son compte. C'est un compte _presque_ normal, toutefois: 
- pour être bien identifiable il a une _carte de visite_ immuable, sans photo et de nom _Comptable_ et ne peut pas se créer des avatars secondaires,
- il a le pouvoir de gérer **les quotas de ressources allouées à l'organisation**:
  - il _partitionne_ les quotas globaux de l'application,
  - pour chaque _partition_ définie, il _délègue_ à certains comptes la capacité à répartir les quotas de la partition aux comptes.

> Le **Comptable** n'a pas plus que les autres comptes les moyens cryptographiques de s'immiscer dans les notes des avatars des comptes et leurs chats: ce n'est en aucune façon un modérateur et il n'a aucun moyen d'accéder aux contenus, pas plus qu'à l'identité des avatars, sauf bien entendu de ceux qu'il connaît personnellement.

### Les comptes autonomes "A"
Un compte _autonome_:
- **paye son abonnement** (qu'il fixe lui-même) **et sa consommation** (sans limite a priori),
- **ne peut être ni _restreint_, ni _bloqué_** tant que son solde est créditeur.

### Les comptes de l'organisation "O"
Pour certaines organisations, les comptes "A" ne sont pas acceptables:
- si un compte "A" quitte l'organisation ou qu'il n'est plus jugé opportun, il ne peut pas être bloqué tant que son solde est créditeur, ce qui peut être sans limite.
- l'organisation peut avoir inclus l'abonnement et la consommation de certains comptes dans ses _frais d'adhésion_ ou équivalents et de ce fait vouloir maîtriser les ressources utilisées par les comptes.

Pour répondre à ces objectifs, il existe une seconde catégorie de compte: **les comptes "O", de l'organisation**.

L'organisation supporte de facto les coûts d'abonnement et de consommation pour le compte mais en contrepartie,
- elle lui fixe des **quotas** d'abonnement et de consommation contraignants,
- elle peut restreindre voire bloquer le compte.

Il peut être compliqué pour le Comptable de gérer les quotas de _tous_ les compte:
- beaucoup d'organisations ont une structure décentralisée en sous-organisations géographiques ou thématiques ayant une part d'autonomie de gestion de leurs adhérents.
- il existe aussi des structures _clanique_, où un ou quelques représentants du clan ont toute liberté pour sponsoriser de nouveaux comptes et leur distribuer des quotas. Le cas échéant un protocole, non géré par l'application, définit comment chaque clan participe aux frais d'hébergement sur la base des quotas qui lui sont attribués.

#### Le Comptable peut _partitionner_ ses quotas globaux d'espace et de calcul
Il peut réserver des quotas pour l'ensemble des comptes "A".

Il peut aussi créer des _partitions_ de l'espace global,
- attribuer à chaque partition des quotas spécifiques d'espace et de calcul,
- confier chaque partition à un ou des _délégués_, leur laissant la possibilité d'attribuer des quotas à chaque compte attachés à leur partition.

### Paiements et dons
> Un compte _augmente son solde_ en faisant parvenir des _paiements_ que le Comptable va enregistrer sans qu'il puisse faire le rapprochement entre un _paiement_ et le compte crédité. 

> Un compte peut _effectuer des dons_ à d'autres comptes à condition de rester créditeur.

> Un compte "O" peut vivre sans effectuer de paiements et des dons, mais il peut aussi disposer d'un solde créditeur et effectuer des dons à des comptes "A" amis.

[Information détaillée à propos de la gestion des comptes](./comptes.html)

[Créer, gérer et supprimer ses avatars](./avatars.html)

## Notes personnelles

**Une note porte un texte** d'au plus 5000 caractères pouvant s'afficher avec un minimum de _décoration_, gras, italique, titres, listes ... Ce texte est modifiable.

**Des fichiers peuvent être attachés à une note**
- les types de fichiers les plus usuels (`.jpg .mp3 .mp4 .pdf ...`) s'affichent directement dans le navigateur, les autres sont téléchargeables.
- il est possible d'ajouter et de supprimer des fichiers attachés à une note.
- quand plusieurs fichiers portent le même _nom_ dans la note, ils sont vus comme des _révisions_ successives, qu'on peut garder, ou ne garder que la dernière, ou celles de son choix.

### Vue hiérarchique: note _parent_ d'un note
- les notes apparaissent à l'écran sous forme _hiérarchique_, une note _parent_ ayant en dessous d'elle des notes _enfants_ (ou aucune).
- les notes n'ayant pas de note _parent_ apparaissent rattachées à celui des avatars du compte a qui elle appartient: cet avatar est une _racine_ de la hiérarchie des notes.

> Un avatar peut créer des notes **personnelles**, les mettre à jour, les supprimer et **les indexer par des mots clés personnels**. Elles sont cryptées, comme toutes les données des comptes, et seul le titulaire du compte a, par l'intermédiaire de sa phrase secrète, la clé de cryptage apte à les décrypter.

[Plus de détails ici à propos des notes personnelles](./notes.html)

## Contacts

Un contact est _un avatar_ dont le compte connaît **la carte de visite** cryptée. 

### Contact _permanent_
Un contact _permanent_ est établi entre deux avatars quand ils ont ouvert un _chat_ entre eux (un _chat_ ne peut pas être supprimé, mais on peut se dispenser de voir ceux _indésirables_). La clé de la carte de visite a été échangée à la création du _chat_.

### Contact _temporaire_
Lorsque **deux avatars sont membres d'un même groupe** (voir un peu plus avant les _groupes_), les clés de leur cartes de visites sont inscrites dans le groupe et sont ainsi accessibles à tous les membres (du moins à ceux ayant droit d'accès aux membres). 

**Ce contact est temporaire**, dure tant que les deux avatars sont membres du groupe.
- un tel contact _disparaît_ quand l'avatar correspondant est résilié du groupe ou disparaît.
- si l'un des deux avatars souhaite rendre ce contact **permanent** il ouvre un _chat_ avec l'autre.

> Un compte peut rester totalement isolé et n'avoir _aucun contact_ avec les autres: à la création de son compte par _sponsoring_, le sponsor comme le sponsorisé peuvent déclarer vouloir ou non **ouvrir un _chat_** entre eux (ce qui les rend _contacts mutuels permanent_).

> Un contact est réciproque, si Julie a Émilie dans ses contacts, Émilie a Julie dans ses contacts: chacun a échangé avec l'autre la clé de cryptage qui permet de lire la carte de visite de l'autre.

### Commentaire personnel et _hashtags_ attachés à ses _contacts_ 
Ils ont pour but de faciliter le filtrage dans le répertoire de ses _contacts_. Le commentaire et les hashtags attachés à un contact, sont spécifiques du compte, lui seul peut les décrypter. La _carte de visite_ d'un contact pouvant évoluer selon la volonté de ce dernier, conserver un nom / commentaire personnel à son propos est une bonne idée.

> Les _hashtags_ attribués par un compte à un contact lui permettent de le classer comme _expert_ _oubliette_ ou _ami_, de s'en servir comme filtre du répertoire de ses contacts et récupérer par exemple tous les _expert_ sauf les _ami_ ...

## "Chats" entre avatars

Un chat peut être **ouvert**,
- à la création du compte entre sponsor et sponsorisé si tous deux en sont d'accord,
- avec le membre _actif_ d'un groupe pour lequel on a droit d'accès aux membres (voir plus loin),
- par contact direct avec une _phrase de contact_.

Les deux avatars peuvent écrire des textes courts:
- un texte d'un _chat_ ne peut plus y être modifié mais peut être supprimé par son auteur,
- le volume total des textes sur le _chat_ est limité à 5000 signes, les plus anciens étant perdus en cas de dépassement de cette limite.

Une fois créé un _chat_ ne disparaît que quand les deux avatars qui le partage ont disparu.
- pour ne pas être importuné, l'un des 2 peut _déclarer le chat indésirable_, ce qui en efface le contenu pour lui. Le _chat_ n'est plus décompte plus pour lui dans son nombre de documents. L'autre peut toujours continuer à y écrire des textes sans être certain d'être lu ... Le _chat_ cesse d'être _indésirable_ dès que l'avatar qui l'a déclaré tel y écrit un texte (et compte à nouveau dans son décompte de documents).
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
- facultativement **des notes partagées entre les membres** qui peuvent les lire et les éditer.

Un avatar connu dans un groupe peut avoir plusieurs états successifs:
- **simple contact**: il a été inscrit comme contact du groupe le sait et peut voir la _carte de visite_ du groupe.
- **contact invité**: un membre actif ayant pouvoir d'animateur a invité le contact à devenir membre actif. L'avatar invité voit cette invitation et s'il l'accepte deviendra membre _actif_, sinon il retournera à l'état de simple contact. Nul ne devient membre actif à son insu.
- **membre actif**: il peut participer à la vie du groupe.

Par défaut le mode d'invitation dans un groupe est **simple**: il suffit **qu'un** animateur invite un contact pour que ce dernier soit invité.
- un second mode dit **unanime** peut être fixé: dans ce cas il faut que **tous** les animateurs aient invité un contact pour qu'il soit effectivement invité.
- en mode unanime, le cercle fermé des animateurs contrôle strictement qui peut être invité. Un _couple_ ne peut ainsi pas devenir un _trouple_ sans l'accord des deux membres du couple.

### Accès aux membres et / ou aux notes
Un membre actif _peut_ recevoir lors de son invitation deux _droits_:
- **droit d'accès aux autres membres** et au _chat du groupe_ (ou non),
- **droit d'accès aux notes** en lecture, en lecture et écriture ou pas du tout.

Lors de son invitation il peut aussi recevoir le **pouvoir d'animation**. S'il ne l'a pas, un membre _animateur_ peut lui conférer ce pouvoir (mais ne pourra plus lui enlever).

> **Certains groupes peuvent être créés à la seule fin d'être un répertoire de contacts** cooptés par affinité avec possibilités de _chat_. Personne n'y lit / écrit de notes.

> **Certains groupes peuvent être créés afin de partager des notes _anonymes_ de discussion**. Par exemple un animateur est seul à avoir droit d'accès aux membres, à les connaître: les notes sont de facto anonymes pour les autres membres.

En général les groupes sont créés avec le double objectif de réunir des avatars qui se connaissent mutuellement, échangent sur le chat et partagent des notes.

### Commentaire personnel et _hashtags_ attachés à un groupe
Tout membre actif peut attacher un commentaire personnel et ses propres _hashtags_ à un groupe. Personne d'autre n'en a connaissance, son commentaire et ses _hashtags_ restent strictement privés et cryptés.

La recherche d'un groupe quand on est membre de beaucoup de groupes en est facilitée.

### Notes d'un groupe
- elles sont cryptées par la clé générée pour le groupe à sa création et qui a été transmise à chaque membre lors de son invitation au groupe.
- hormis les membres actifs du groupe ayant droit d'accès aux notes, personne ne peut accéder aux notes du groupe.
- quand un nouveau membre accepte une invitation au groupe avec droits d'accès aux notes, il a immédiatement accès à toutes les notes existantes du groupe. S'il redevient _simple contact_ ou perd son droit d'accès aux notes (de par sa volonté ou celle d'un _animateur_), il n'a plus accès à aucune de celles-ci (ce qui allège ses sessions).
- pour écrire / modifier / supprimer une note du groupe, il faut avoir le droit d'accès en écriture aux notes.
- chaque note détient la liste des membres qui y sont intervenus en mise à jour.

### _Hashtags_ d'un membre à une note d'un groupe
Chaque membre d'un groupe peut attacher à une note ses _propres hashtags_ que les autres membres ne sont pas en mesure de décrypter. Ceci facilite les recherches d'un compte dans ses notes, le filtrage par _hashtags_ s'effectuant sur l'ensemble des notes auxquelles le compte a accès.

Par ailleurs, un _animateur_ peut attacher des _hashtags publics_ à une note, lisibles de tous les membres et seulement d'eux.

### Vue hiérarchique: note de groupe _parent_ d'une autre note du même groupe
Ceci fait apparaître visuellement à l'écran une hiérarchie.

**Une note personnelle peut avoir pour _parent_ une note de groupe** pour la compléter / commenter: toutefois l'avatar propriétaire de la note personnelle sera seul à la voir (puisqu'elle est _personnelle_).

### Membre _hébergeur_ d'un groupe
_L'hébergeur du groupe_ est un membre qui s'est dévoué pour supporter les coûts d'abonnement de stockage (nombres de notes et volume des fichiers) des notes du groupe mais peut en limiter le nombre / volume.

[Information détaillée à propos des groupes et de leurs notes](./notes.html)

# Modes de connexion *synchronisé*, *incognito* et *avion*

Pour se connecter à son compte, le titulaire d'un compte choisit sous quel **mode** sa session va s'exécuter: _synchronisé_, _avion_ ou _incognito_.

### Mode "normal" _synchronisé_ 
C'est le mode préférentiel où toutes les données du périmètre d'un compte sont stockées dans une micro base locale cryptée dans le navigateur remise à niveau depuis le serveur central à la connexion d'une nouvelle session puis maintenue à jour en cours de session.

Une connexion ultérieure après une session synchronisée est rapide et économique: l'essentiel des données étant déjà dans le navigateur, seules les _mises à jour_ sont tirées du serveur central.

### Mode _avion_
Pour que ce mode fonctionne il faut qu'une session antérieure en mode _synchronisé_ ait été exécutée dans ce navigateur pour le compte. A la connexion le titulaire du compte y voit l'état dans lequel étaient ses données à la fin de sa dernière session synchronisée dans ce navigateur. **L'application ne fonctionne qu'en lecture**.

> Avant d'appeler l'application, on peut couper le réseau (le mode _avion_ sur un mobile), de façon à ce que même l'ouverture de la page de l'application ne cherche pas à vérifier si une version plus récente est disponible.

### Mode _incognito_
**Aucun stockage local n'est utilisé, toutes les données viennent du serveur central**, l'initialisation de la session est plus longue qu'en mode synchronisé. Aucune trace n'est laissée sur l'appareil (utile au cyber-café ou sur le mobile d'un.e ami.e).

> On peut ouvrir l'application dans une _fenêtre privée_ du navigateur, ainsi même le texte de la page de l'application sera effacé en fermant la fenêtre.

> **En utilisant des sessions synchronisées sur plusieurs appareils, on a autant de copies synchronisées de ses notes et chats sur chacun de ceux-ci**, et chacun peut être utilisé en mode avion.

[En savoir plus sur les modes, l'accessibilité des fichiers en mode _avion_, le _presse-papier_](./modessync.html)

# Coûts d'hébergement de l'application

Le coût d'usage de l'application pour une organisation correspond aux coûts d'hébergement des données et de traitement de celles-ci. Selon les techniques et les prestataires choisis, les coûts unitaires varient mais existent dans tous les cas.

### _Base de données_ et _Storage_
- la _bases de données_ enregistre toutes les données de l'application **SAUF** les fichiers attachés aux notes. Elle requiert des accès rapides.
- les fichiers attachés aux notes sont enregistrés dans un _Storage_, stockage distant ayant une gestion spécifique et économique du fait d'être soumis à peu d'accès mais de plus fort volume.

Leur stockage ont des coûts unitaires très différents (variant d'un facteur de 1 à 6).

### Abonnement: coût de l'espace occupé en permanence
L'abonnement couvre les frais fixes d'un compte: même quand il ne se connecte pas, le stockage de ses données a un coût. Il est décomposé en deux lignes de coûts correspondant à l'occupation d'espace forfaitaire en _base de données_ et en _storage_:
- _Coût 1_ : **prix unitaire de stockage d'un document** multiplié par le **le nombre maximal du quota de _documents_ de l'abonnement**: notes personnelles et notes d'un groupe hébergé par le compte, chats personnels, nombre de participations actives aux groupes.
- _Coût 2_ : **prix unitaire du stockage des fichiers dans un _storage_** multiplié par le **le volume maximal du quota de l'abonnement des fichiers attachés aux notes**.

> Pour obtenir le coût correspondant à ces deux volumes il est pris en compte, non pas _le volume effectivement utilisé à chaque instant_ mais forfaitairement **les _quotas_ (_volumes maximaux_)** auquel le compte est abonné.

> Les volumes _effectivement utilisés_ ne peuvent pas dépasser les quotas (volumes maximum) déclarés pour l'abonnement. Dans le cas où un changement de l'abonnement réduit a posteriori ces maximum en dessous des volumes utilisés, les volumes n'auront plus le droit de croître.

### Consommation : coût de calcul et de transfert des fichiers
Le coût de calcul apparaît lors de l'usage effectif de l'application quand une session d'un compte est ouverte. Elle comporte 4 lignes:
- **nombre _de lectures_** (en base de données).
- **nombre _d'écritures_** (en base de données).
- **volume _descendant_** (download) de fichiers téléchargés en session depuis le _storage_.
- **volume _montant_** (upload) de fichiers envoyés dans le _storage_ pour chaque création / mise à jour d'un fichier.

### Coûts réel et facturé
Le coût _réel_ d'un compte sur une période donnée correspond à la somme du coût _d'abonnement_ et du coût de _consommation_ pendant cette période.

> Le coût _facturé_ sur une période est nul pour un compte "O" et est le coût _réel_ pour un compte "A".

Le solde d'un compte est calculé à tout instant,
- en le créditant des paiements et dons reçus dans la période,
- en le débitant des dons effectués et des coûts _facturés_ dans la période.

> Les tarifs peuvent changer d'un mois à l'autre, mais sont fixes dans un mois donné.

Un compte peut afficher à tout instant,
- l'état de sa comptabilité **calculée à l'instant de l'affichage**: quotas de l'abonnement, nombre de documents et volumes de fichiers effectivement occupés, **solde courant**.
- l'historique de l'évolution mois par mois sur les 12 derniers mois: moyennes des quotas, des consommations, coût total sur le mois (réel et facturé), débits et crédits, soldes en début et fin de mois.

> L'unité monétaire interne est le _centime_ (c). Son ordre de grandeur est voisin d'un centime d'euro en appliquant des tarifs proches des coûts _de base_ des hébergements sur le marché. Chaque fournisseur a ses propres tarifs qui peuvent être significativement éloignés des coûts _de base_.

>_L'ordre de grandeur_ pour un compte varie en gros de **0,5€ à 3€ par an**. Individuellement ça paraît faible mais n'est plus du tout négligeable pour une organisation assurant les frais d'hébergement d'un millier de comptes ...

### Gestion des _abonnements gratuits_ des comptes "O"
Le principe est pour le Comptable de confier la distribution fine des quotas à des _délégués_ responsables eux-mêmes de les répartir entre les comptes qui dépendent d'eux en procédant à un _découpage en partitions_ des ressources globales dont il dispose. Chacune _partition_ est dotée de trois quotas, 
- un quota QN de _nombre de documents_ qui ne doit pas être dépassé, 
- un quota QV de _volume de fichiers_ qui ne doit pas être dépassé,
- un quota QC de _coût calcul_: c'est en _centime_ le coût calcul maximal sur un mois (calculé et moyenné sur M / M-1).

Tout compte "O" est attaché à une _partition_ à son sponsoring, le Comptable seul pouvant ensuite éventuellement le basculer dans une autre _partition_.

Pour chaque _partition_ le Comptable _peut_ confier une _délégation_ à certains comptes de la partition afin que ceux-ci fixent aux comptes "O" de leur partition leurs quotas personnels QN / QV / QC.

> Remarque: un compte "A" se fixe lui-même ses 3 quotas QN / QV / QC et en assume les conséquences sur ses coûts facturés.

[En savoir plus sur les partitions des comptes "O"](./partitions.html)

## Alertes et restrictions d'accès des comptes
### Alertes de _comptabilité des coûts_
Le décompte des coûts est susceptible de **lever des alertes** à chaque compte en fonction de sa situation personnelle.

#### Coût de calcul excédant son quota (QC): ralentissements
Les coûts de calcul relevés sur le mois en cours et le précédent sont ramenés à un _mois de 30 jours_. Il y a excès quand ce coût moyen de calcul excède le quota QC.
- les opérations sont artificiellement ralenties par un délai d'attente d'autant plus grand que l'écart avec le quota est fort.
- les transferts de fichiers le sont également selon le même coefficient mais proportionnellement au volume du fichier.

> Remarque: un compte "A" étant lui-même maître de son quota QC peut éviter les ralentissements en augmentant fortement son quota QC: ce faisant il inhibe une alerte de surconsommation de calcul qui peut s'avérer coûteuse pour lui.

> Remarque: à tout instant un compte peut voir le nombre de jours estimé en solde positif en considérant la prolongation de sa situation actuelle (mêmes quotas, même consommation moyenne de calcul).

#### Nombre de documents excédant son quota (QN): blocage de l'ajout de documents
Le compte ne peut pas créer de nouvelles notes, de chats ou participer à de nouveaux groupes. Il est ainsi invité à,
- supprimer des notes,
- déclarer des chats _indésirables_,
- se résilier de certains groupes,
- OU ... demander une augmentation de son quota QN au Comptable ou à un de ses délégués si c'est un compte "O", procéder lui-même à cette augmentation si c'est un compte "A".

#### Volume de fichiers attachés aux notes excédant son quota (QV): blocage du volume
Le compte ne peut pas ajouter de fichiers ou remplacer un fichier par un autre de taille supérieure. Il est ainsi invité à,
- supprimer des fichiers et/ou des révisions de fichiers,
- remplacer les gros fichiers par des révisions plus compressés,
- OU ... demander une augmentation de son quota QV au Comptable ou à un de ses délégués si c'est un compte "O", procéder lui-même à cette augmentation si c'est un compte "A".

#### Solde négatif: le compte est mis en ACCÈS RESTREINT
Le compte ne peut plus ni lire ses données, ni les mettre à jour. Toutefois il conserve les possibilités de:
- visualiser ses alertes et sa comptabilité,
- effectuer des versements et les enregistrer afin que le Comptable les créditent,
- recevoir des dons,
- discuter de sa situation sur les _chats d'urgence_ avec le Comptable et ses _délégués_ pour un compte "O", voire de bénéficier d'un _don_ (à rembourser plus tard le cas échéant).

## Alerte de l'Administrateur Technique à l'organisation
**Cette alerte porte toujours un message d'information** (_arrêt programmé ..._).

Elle _peut_, soit ne porter aucune restriction, soit porter l'une de celles-ci:
- **Espace figé**. Aucune écriture ne peut être faite, typiquement afin de procéder à une opération technique d'export, verrouillage d'une archive d'un espace ... mais peut aussi être une mesure de rétorsion.
- **Espace clos**. L'Administrateur Technique a effacé les données de l'espace dont il ne subsiste plus que cette alerte dont le texte donne la raison et le cas échéant indique si l'espace est accessible par une autre URL.
- **Connexions bloquées à partir d'une date d**: les comptes ne seront pas détruits à cette date mais plus aucune connexion ne pourra être faite à partir de d.

### Alerte du _Comptable_ ou de ses _délégués_ aux comptes "O"
Le Comptable ou ses délégués peuvent inscrire une _alerte_:
- adressée à TOUS les comptes "O" d'une _partition_,
- adressée à UN compte "O" spécifique.

Chaque compte "O" est en conséquence potentiellement la cible de deux alertes.

Une telle alerte **porte un message d'information**, et peut être associé ou non à une restriction.

#### Accès en LECTURE SEULEMENT
Le compte **ne peut que consulter les données** (comme en mode _avion_).

#### ACCÈS RESTREINT
Le compte ne peut plus ni lire ses données, ni les mettre à jour. 

Dans tous les cas il conserve toutefois les possibilités de:
- visualiser ses alertes et sa comptabilité,
- effectuer des versements et les enregistrer afin que le Comptable les créditent,
- discuter de sa situation sur les _chats d'urgence_ avec le Comptable et ses _délégués_ pour un compte "O".

## Date limite de vie d'un compte
Cette date, toujours le dernier jour d'un mois, signifie que le jour suivant le compte sera irrémédiablement détruit sans possibilité de récupération.

Cette date est fixée lorsqu'un compte se connecte: en conséquence le compte en est immédiatement informé.

Quand le compte **N'EST PAS en ACCÈS RESTREINT** elle est fixée à 12 mois plus tard. En d'autres termes le compte a toujours un an de vie après sa dernière connexion.

Quand le compte **EST en ACCÈS RESTREINT** elle est fixée à 6 mois après le jour du basculement du compte en ACCÈS RESTREINT.

A chaque connexion un compte voit la date limite de vie de son compte et le nombre de jours à partir du jour courant. Si ce nombre est faible (inférieur à 30), l'alerte est **rouge**.

> Remarque: un compte n'a de facto que 6 mois de vie assurée après la dernière connexion où il a constaté de na pas être en ACCÈS RESTREINT. Un compte "A" est maître de cette durée en s'assurant que son compte restera créditeur. Un compte "O" peut à l'inverse faire l'objet d'un ACCÈS RESTREINT imposé par le Comptable ou un de ses délégués, et ce en dehors de son contrôle.

[Les alertes et restrictions en détail](./alertes.html).

## Enregistrement de crédits
Afin de préserver l'anonymat dans le _vrai_ monde des comptes, la procédure de _paiement_ et d'enregistrement des crédits s'effectue de la manière suivante.

Le compte, 
- ouvre la fenêtre des paiements et déclare un paiement avec son montant. Il en résulte un _numéro de ticket de paiement_ et un paiement enregistré en attente.
- il fait parvenir au Comptable par le moyen prévu par l'organisation (typiquement un virement d'une banque, mais aussi tout autre procédé accepté par l'organisation) le montant correspondant accompagné du numéro du ticket obtenu ci-avant.

Quand le Comptable reçoit un paiement avec un numéro qui l’anonymise, il enregistre le montant reçu pour le ticket (qu'il trouve dans la liste des tickets en attente).
- si le compte était _en rouge_ pour solde négatif, dès l'enregistrement par le Comptable, l'indicateur s'efface si le crédit est suffisant.
- les tickets enregistrés par le Comptable restent visibles pour lui jusqu'à M+2 de leur création.

> Les _tickets_ étant enregistrés cryptés par la clé des comptes, aucune corrélation ne peut être faite entre la source d'un _paiement_ et le compte qui en bénéficie.

# Statistiques partagées entre le Comptable et l'Administrateur Technique

Tous deux ont besoin d'éléments statistiques de consommation afin d'ajuster si nécessaire les bases de facturation en fonction d'un usage réel.

Ces statistiques sont calculées mensuellement et sont des fichiers CSV.
- elles sont anonymes: les identifiants des comptes concernés n'y figurent pas.
- pour un mois donné elles sont immuables, calculées à un moment où le mois étant terminé, les compteurs mensuels ne sont plus susceptibles de changer.

**Abonnement / consommation des comptes**: cette statistique est importante pour l'Administrateur Technique pour lui donner une vue comparative des quotas (_abonnements_) et des espaces effectivement occupés.

**Archive des _tickets_ de paiement des comptes "A"**: cette statistique n'est accessible que par le Comptable, l'Administrateur Technnique n'en a cure.

[Détail des statistiques](./stats.html)

# Gestion des _espaces_

**L'administrateur technique** d'un site peut y héberger techniquement plusieurs **espaces / organisations**.

Tout ce qui précède se rapporte à UN espace et les utilisateurs d'une organisation n'ont aucune perception des autres espaces hébergés pour d'autres organisations par les mêmes services techniques.
- dans la base de données, les informations sont partitionnées par nom de l'organisation.
- dans l'espace de stockage des fichiers, des sous-espaces sont séparés par nom de l'organisation.

> Certaines organisations peuvent souhaiter avoir plus d'un espace pour elle: par exemple un espace de _production_, un espace de _démonstration / training_, un espace _archive récente_ ...

L'Administrateur Technique a la possibilité d'ouvrir _instantanément_ un nouvel espace pour une organisation en faisant la demande. 
- Le Comptable et l'Administrateur Technique se sont mis d'accord sur le volume utilisable et la participation aux frais d'hébergement.
- Cette ouverture crée une phrase de _sponsoring_ à destination du Comptable de la nouvelle organisation, 
- Le Comptable créé son compte en utilisant cette phrase de _sponsoring_ et en fixant sa phrase secrète.

_Rappel_; l'Administrateur Technique peut,
- émettre une notification d'information visible de tous les comptes,
- interdire toute connexion à partir d'une date donnée,
- bloquer l'espace de l'organisation en _lecture seule_,
- détruire les données par clôture de l'espace ne laissant pendant un certain temps qu'une seule information d'explication.
