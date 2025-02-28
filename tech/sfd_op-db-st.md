---
layout: page
title: Le service OP des "opérations" d'accès à la base
---

Spécifications Fonctionnelles Détaillées du service OP des opérations de mise à jour / consultation de la base de données et du _storage_ des fichiers.

## Concepts structurants : espaces, comptes, GC

### Espaces
Une base de données / service OP, héberge plusieurs espaces étanches entre eux. Un espace est identifié par `org`, **un code d'organisation**.

Tous les documents comportent:
- soit une colonne / attribut `id` de la forme `org@idd` identifiant unique du document, où:
  - `org` est le code de l'organisation détentrice du document: ce code _préfixe_ donc toutes les id.
  - `idd` est l'identifiant du document dans son organisation.
  - pour les documents `espaces` et `syntheses`, l'id est simplement `org`.
- soit un couple de colonnes / attributs `id / ids` identifiant un sous-document `ids` d'un document `id`.

> Les documents `espaces syntheses` ont par convention un string vide une id de document (ce sont des singletons pour une organisation donnée).

> Tous les autres documents ont une colonne / propriété `id` de document string NON vide, et le cas échéant une colonne / propriété `ids` string NON vide. 

#### Code organisation attaché à un espace
A la déclaration d'un espace, l'administrateur technique du site déclare un **code organisation**:
- ce code ne peut plus changer: lors d'une _exportation_ d'un espace on peut définir un autre code d'espace pour la cible de l'importation.
- le Storage de fichiers comporte un _folder_ racine portant ce code d'organisation ce qui partitionne le stockage de fichiers.
- les connexions aux comptes citent ce _code organisation_.

### Comptes

#### L'administrateur technique N'A PAS de compte
Il a pour rôle majeur de gérer les espaces:
- les créer / les détruire,
- définir leurs quotas à disposition du Comptable de chaque espace: il existe trois quotas,
  - `qn` : nombre maximal autorisé des notes, chats, participations aux groupes,
  - `qv` : volume total autorisé des fichiers attachés aux notes.
  - `qc` : quota de calcul mensuel total en unités monétaires.
- ces quotas sont _indicatifs_ sans blocage opérationnel et servent de prévention à un crash technique pour excès de consommation de ressources.

Ses autres rôles fonctionnels sont :
- la gestion d'une _notification / blocage_ par espace, 
  - soit pour information technique importante, 
  - soit pour _figer_ un espace avant sa migration vers un autre (ou sa destruction).
- l'export de la _base_ d'un espace vers une autre,
- l'export des fichiers d'un espace d'un _Storage_ à un autre.
- il peut lire pour chaque espace un fichier CSV relevé mensuel de consommation des comptes (mais sans leur ID) pour comparer ses coûts d'hébergement et ce qu'a reçu le Comptable.

#### LE Comptable de chaque espace
Pour chaque espace, il existe un compte d'id `300000000000` qui est le **Comptable** de l'espace.

#### Comptes _standard_
Il existe deux catégories de comptes:
- **les comptes "O", de l'organisation**, bénéficient de ressources _gratuites_ attribuées par la Comptable et ses _délégués_. En contrepartie de cette _gratuité_ un compte "O" peut être _bloqué_ par le Comptable et ses _délégués_ (par exemple en cas départ de l'organisation).
- **les comptes "A", autonomes**, achètent des ressources sous la forme d'un abonnement et d'une consommation. Tant qu'il est créditeur un compte "A" ne peut pas être bloqué.

> Les abonnements et consommations sont exprimées in fine en _unité monétaire_ virtuelle, le centime (c), dont l'ordre de grandeur est voisin d'un centime d'euro ou de dollar (le _cours_ exact étant fixé par chaque organisation).

#### Comptes "O" : _partitions_
Le Comptable dispose des quotas globaux de l'espace attribués par l'administrateur technique. 
- Il définit un certain nombre de **partitions de quotas**.
- Il confie la gestion de chaque partition à des comptes _délégués_ qui peuvent distribuer des quotas de ressources aux comptes "O" affectés à leur partition.

Un compte "O",
- est attaché à un instant donné, à une seule _partition_: ses quotas `qc q1 q2` sont prélevés sur ceux de sa partition. 
- est créé par _sponsoring_,
  - soit d'un compte "O" existant _délégué_,
  - soit du Comptable qui a choisi de quelle partition il relève.

Les comptes "0" _délégués_ d'une partition peuvent:
- sponsoriser la création de nouveaux comptes "O", _délégués_ eux-mêmes ou non de cette partition.
- gérer la répartition des quotas entre les comptes "O" attachés à cette partition.
- gérer une _notification / blocage_ pour les comptes "O" attachés à leur partition.

#### Comptes "A"
Un compte "A" est créé par _sponsoring_,
- soit d'un compte "A" existant qui à cette occasion fait _un don_ au compte sponsorisé d'un montant de son choix prélevé sur son solde monétaire.
- soit par un compte "O" _délégué_ ou par le Comptable: un don de bienvenue peut aussi être effectué.

Un compte "A" définit lui-même ses quotas `qn` et `qv` (il les paye en tant _qu'abonnement_) et n'a pas de quotas `qc` (il paye sa _consommation_).

#### Rôles du Comptable
Le rôle principal d'un _Comptable_ est,
- de partitionner les quotas globaux attribuables aux comptes "O" et d'ajuster les quotas de chaque partition,
- de désigner les _délégués_ de chaque partition, le cas échéant de retirer ou d'ajouter la qualité de _délégué_ a un compte "O" de la partition.
- de changer un compte "O" de partition.
- de gérer des _notifications / blocages_ s'appliquant à des comptes "O" spécifiques ou à tous les comptes d'une partition.
- d'enregistrer les paiements des comptes.
- de sponsoriser directement la création de nouveaux comptes.

Le Comptable est un compte "O" qui:
- ne peut pas se résilier lui-même,
- ne peut pas changer de partition de quotas, il est rattaché à la première partition de son espace qui ne peut pas être supprimée.
- ne peut pas supprimer son propre attribut _délégué_,
- accepte l'ouverture de **chats** avec n'importe quel compte "O" ou "A" qui en prend l'initiative.

### GC _garbage collector_** 
C'est un traitement de nettoyage qui est lancé une fois par jour. Il a plusieurs fonctionnalités techniques:
- suppression de rows / documents obsolètes,
- détection des comptes à détruire par inutilisation,
- détection des groupes non hébergés depuis un certain temps et suppression de ceux-ci,
- nettoyage des fichiers fantômes sur _Storage_,
- calcul de _rapports / archivages_ mensuels pouvant conduire à purger des données vivantes de la base.

C'est un service externe de **CRON** qui envoie journellement une requête de GC à une instance de service OP.

# Base de données accédée par le service OP

L'organisation diffère entre bases SQL (SQLite, PostgrSQL ultérieurement) et NOSQL (Firestore).
- **SQL** - Les données sont distribuées dans des **tables** `espaces avatars versions notes ...` contenant des **rows**, chacun ayant plusieurs **colonnes**.
- **NOSQL** - Chaque table SQL correspond à une **collection de documents**, chaque **document** est équivalent à un row de la table SQL de même nom que la collection.

Les _colonnes_ d'une table SQL correspondent aux _attributs / propriétés_ d'un document.
- en SQL la _clé primaire_ est une colonne `id` ou un couple de colonnes `id / ids`,
- en Firestore le _path_ d'un document contient cette propriété `id` ou couple de propriétés `id / ids`.

## Attributs des tables / documents
Ils sont listés dans un objet de configuration:

    static _attrs = {
      espaces: ['id', 'v', 'dpt', '_data_'],
      fpurges: ['id', '_data_'],
      partitions: ['id', 'v', '_data_'],
      syntheses: ['id', 'v', '_data_'],
      comptes: ['id', 'v', 'hk', '_data_'],
      comptis: ['id', 'v', '_data_'],
      invits: ['id', 'v', '_data_'],
      comptas: ['id', 'v', 'dlv', '_data_'],
      versions: ['id', 'v', 'dlv'],
      avatars: ['id', 'v', 'vcv', 'hk', '_data_'],
      notes: ['id', 'ids', 'v', '_data_'],
      transferts: ['id', 'dlv', '_data_'],
      sponsorings: ['id', 'ids', 'v', 'dlv', '_data_'],
      chats: ['id', 'ids', 'v', '_data_'],
      tickets: ['id', 'ids', 'v', 'dlv', '_data_'],
      groupes: ['id', 'v', 'dfh', '_data_'],
      membres: ['id', 'ids', 'v', '_data_'],
      chatgrs: ['id', 'ids', 'v', '_data_']
    }

### Remarques
- `id` et `hk` sont, en base, de la forme org@id et org@hk. Pour espaces et syntheses simplement de la forme org.
- `hk`: clés d'accès _externes_, hash de phrase secrète ou de contact / sponsoring.
- `ids` sont des clés secondaires relatives à la clé majeure id. Dans le seul cas de `sponsorings`, `ids` peut être utilisé comme clé alternative d'accès.
- `v vcv`: sont des entiers, numéros de versions en séquence contine croissante.
- `dpt dlv dfh`: sont entiers représentant des dates `aaaammjj` (`20250301`).
- `_data_`: sont des objets sérialisés contenant les données du documents, dont ses attributs _externalisés_ ci-avant.
  - `versions` n'a pas de `_data_` en base, celui-ci étant reconstruit à la lecture de ses attributs externes `{id, v, dlv}`.

### Cryptage en base
Les attributs _data_ sont toujours cryptés en base, ce n'est pas optionnel.

Les autres attributs sont _externalisés_,
- soit parce que servant de clé primaire d'accès (id ids),
- soit parce que servant de clé alternative d'accès (hk),
- soit parce que nécessaire comme valeur de filtrage (v vcv dpt dlv dfh).

Pour chaque choix d'un site:
- org peut être crypté ou non.
- id peut être crypté ou non.
- ids peut être crypté ou non.

> Le cryptage de `org@id` est `corg@cid`, les deux termes `org` / `id` étant ou non cryptés séparément en `corg` / `cid`.

Les attributs `v vcv dpt dlv dfh` ne sont PAS cryptés, étant utilisés par comparaison d'ordre et pas seulement d'égalité.

> Aucun _meta-lien_ n'est accessible à la lecture d'une base cryptée, les codes des organisations hébergés comme les id des comptes / avatars / groupes ... étant également obfusqués.

# Tables / collections _techniques_ de nettoyage du _Storage_

L'écriture et la suppression de fichiers du _Storage_ ne sont pas soumises à la gestion transactionnelle du commit de la base. En conséquence, elles peuvent être,
- **marqués dans la base comme _en écriture sur Storage_**, puis une fois vraiment écrits, enregistrés comme tels dans la base. Mais si un problème technique intervient au mauvais moment, ils peuvent ne pas être marqués _écrits_ dans la base et être cependant présents dans le _Storage_: il faut périodiquement nettoyer ces fichiers _fantômes_ dont l'upload n'a pas été enregistré en base. C'est l'objet des documents `transferts`.
- **considérés comme _logiquement_ détruits dans la base**, mais n'ayant pas encore été physiquement purgés du _Storage_. Il faudra, un jour, achever d'effectuer ces _purges_ physiques du _Storage_. C'est l'objet des documents `fpurges`.

## Documents `transferts`
- `id`:
- `dlv`:

- `_data_`: sérialisation de:
  - `id`: `avgrid + '_' + idf`,
  - `avgrid`: id de l'avatar / groupe propriétaire du fichier.
  - `idf`: identifiant du fichier.
  - `dlv`: date-limite de validité, lendemain du jour d'écriture (début de _l'upload_).

Ces documents ne sont jamais mis à jour une fois créés, ils sont supprimés,
- en général quasi instantanément dès que _l'upload_ est physiquement terminé,
- sinon par le GC qui considère qu'un upload ne peut pas techniquement être encore en cours à j+2 de son jour de début.

## Documents `fpurges`
- `id`: id aléatoire générée à la création,

- `_data_` : liste encodée,
  - `avgrid` d'un avatar ou d'un groupe, correspondant à un folder du _Storage_ à supprimer,
  - `lidf`: array des idf des fichiers à purger.

Ces documents ne sont jamais mis à jour une fois créés, ils sont supprimés par le prochain GC après qu'il ait purgé du _Storage_ tous les fichiers cités dans _data_.

# Table / documents entête d'un espace: `espaces syntheses`

Pour un espace donné, `A`, ce sont des singletons:
- `espaces`: `id` est un string vide. Contient les données de l'espace.
- `syntheses`: `id` est est un string vide. Le document contient des données statistiques sur la distribution des quotas aux comptes "O" (par _partition_) et l'utilisation de ceux-ci.

> Leur clé d'accès en base est donc `org`, leur organisation.

# Tables / collections _majeures_ : `partitions comptes comptis invits comptas avatars groupes`

Chaque collection a un document par `id` (clé primaire en SQL, second terme du path en Firestore).

### `partitions`
Un document par _partition de quotas_ décrivant la distribution des quotas entre les comptes "O" attachés à cette partition.
- `id` est un id aléatoire `2...`.
- Clé primaire : `org@2...`. Path : `partitions/org@2...`

### `comptes`
Un document par compte donnant les clés majeures du compte, la liste de ses avatars et des groupes auxquels un de ses avatars participe. `id`, le numéro du compte, est un string aléatoire commençant par `3` :
- `300000000000` : pour le Comptable.
- `3...` : pour les autres comptes.
- Clé primaire : `org@3...`. Path : `comptes/org@3...` 

### `comptis`
Un document _complémentaire_ de `comptes` (même `id`) qui donne des commentaires et hashtags attachés par le comptes aux avatars et groupes de sa connaissance.

### `invits`
Un document _complémentaire_ de `comptes` (même `id`) qui donne la liste des invitations aux groupes pour les avatars du compte et en attente d'acceptation ou de refus.

### `comptas`
Un document _complémentaire_ de `comptes` (même `id`) donnant ses compteurs de consommation et les quotas.

### `avatars`
Un document par avatar donnant les informations d'entête d'un avatar. 
`id` est un string aléatoire commençant par `3`
- Clé primaire : `org@3...`. Path : `comptes/org@3...` `avatars/org@3...`

### `groupes`
Un document par groupe donnant les informations d'entête d'un groupe. 
`id` est un string aléatoire commençant par `4`.
- Clé primaire : `org@4...`. Path : `groupes/org@4...`

# Tables / sous-collections d'un avatar ou d'un groupe
- chaque **avatar** a 4 sous-collections de documents: `notes sponsorings chats tickets` (seul l'avatar Comptable a des tickets).
- chaque **groupe** a 3 sous-collections de documents: `notes membres chatgrs`.

Dans chaque sous-collection, `ids` est un identifiant relatif à `id`. 
- en SQL les clés primaires sont `org@id ids`
- en Firestore les paths sont (par exemple pour la sous-collection `notes`) : `versions/org@id/notes/ids`, `id` est le second terme du path, `ids` le quatrième.

### `notes`
Un document représente une note d'un avatar ou d'un groupe. L'identifiant relatif `ids` est un string aléatoire. 

### `sponsorings`
Un document représente un sponsoring d'un avatar. Son identifiant relatif `ids` est _hash de la phrase_ de sponsoring entre le sponsor et son sponsorisé et est utilisé comme clé alternative d'accès.

### `chats`
Un chat entre 2 avatars I et E se traduit en deux documents : 
- l'un sous-document de I a pour identifiant secondaire `ids` un string aléatoire.
- l'autre sous-document de E a pour identifiant secondaire `ids` un string aléatoire.
- chacun des deux chats a connaissance de `id / ids` de l'autre.

### `membres`
Un document par membre avatar participant à un groupe. L'identifiant secondaire `ids` est l'indice membre `1..N`, ordre d'enregistrement dans le groupe.

### `chatgrs`
Un seul document par groupe. `id` est celui du groupe et `ids` vaut toujours `1`.

### `tickets`
Un document par ticket de crédit généré par un compte A. `ids` est un nombre aléatoire tel qu'il puisse s'éditer sous forme d'un code à 6 lettres majuscules (de 1 à 308,915,776).

# Phrases secrètes et clés de cryptage
## Phrases
### Phrase de création du comptable
- CC : PBKFD de la phrase complète - hCC son hash.
- CR : PBKFD d'un extrait de la phrase - hCR son hash.

### Phrase secrète d'accès à un compte
- XC : PBKFD de la phrase complète - hXC son hash.
- XR : PBKFD d'un extrait de la phrase - hXR son hash.

### Phrase de sponsoring
- YC : PBKFD de la phrase complète - hYC son hash
- YR : PBKFD d'un extrait de la phrase - hYR son hash.

### Phrase de contact d'un avatar
- ZC : PBKFD de la phrase complète - hZC son hash.
- ZR : PBKFD d'un extrait de la phrase - hZR son hash.

## Clés
### S : clé du site
Elle est fixée dans la configuration de déploiement des services OP et PUBSUB par l'administrateur technique.
- **elle crypte les _data_ des documents**, c'est à dire l'ensemble des propriétés d'un document. Les propriétés externalisées en index / clé sont répliquées en clair en dehors de _data_.

### E : clé d'un espace
- attribuée à la création de l'espace par l'administrateur.
- clé partagée entre l'administrateur et le Comptable de l'espace.
- **crypte les rapports générés par le GC** de ce fait lisibles pour l'administrateur et le Comptable.

### K : clé principale d'un compte.
- attribuée à la création du compte par `AccepterSponsoring` ou `CreerEspace` pour le Comptable.
- propriété exclusive du compte.
- crypte ses notes et d'autres clés.

### A : clé d'un avatar
- attribuée à la création de l'avatar (du compte pour l'avatar principal).
- **crypte les photo et texte de sa carte de visite**.
- crypte la clé G d'un groupe auquel l'avatar est invité.

### C : clé d'un chat
- attribuée aléatoirement à la création du chat.
- **crypte les textes du chat**.

### G : clé d'un groupe
- attribuée à la création du groupe.
- crypte les photo et texte de sa carte de visite, ses notes, les textes du chat du groupe.
- crypte les clés A des membres du groupe.

### P : clé d'une partition
- attribuée à la création de la partition par le Comptable et à la création de l'espace pour la partition primitive.
- crypte les textes des notifications de la partition et les clés A des avatars principaux des comptes de la partition.

## Clé RSA d'un avatar
La clé de cryptage (publique) et celle de décryptage (privée) sont de longueurs différentes. 

Le résultat d'un cryptage a une longueur fixe de 256 bytes. Deux cryptages RSA avec la même clé d'un même texte donnent deux valeurs cryptées différentes.

Chaque avatar a un couple de clés privée / publique:
- la clé privée est stockée cryptée par la clé K du compte dans le document `avatars` et pour l'avatar principal seulement elle est redondée dans le document `comptes`.
- la clé publique est stockée en clair dans le document `avatars`.

## Documents stockant les clés, phrases et hash de phrases
### `espaces`
- `cleES` : clé E cryptée par la clé S.

### `comptes`
- `hXC`: hash du PBKFD de la phrase secrète complète.
- `hk`: `hXR` hash du PBKFD d'un extrait de la phrase secrète.
- `cleKXC` : clé K cryptée par XC.
- `cleEK` : Comptable seulement. Clé E cryptée par sa clé K.
- `privK` : clé privée RSA de son avatar principal cryptée par la clé K du compte.
- `cleAK` : _pour chaque avatar du compte_:  clé A de l'avatar cryptée par la clé K du compte.
- `cleGK` : _pour chaque groupe_ où un avatar est actif: clé G du groupe cryptée par la clé K du compte.
- _Comptes "O" seulement:_
  - `clePK` : clé P de la partition cryptée par la clé K du compte. Toutefois si cette clé a une longueur de 256, la clé P peut être décryptée par `privK`, ayant été cryptée par la clé publique de l'avatar principal du compte suite à une affectation à une partition APRÈS sa création (changement de partition, passage de compte A à O).

### `invits`
- `cleGA` : _pour chaque groupe_ où l'avatar est invité.

### `avatars`
- `cleAZC` : clé A cryptée par ZC.
- `pcK` : phrase de contact cryptée par la clé K du compte.
- `hZC` : hash du PBKFD de la phrase de contact complète.
- `hk` : `hZR` hash du PBKFD d'un extrait de la phrase de contact.
- `pub privK` : couple des clés publique / privée RSA de l'avatar.

### `sponsorings`
- `ids`: `hYR` hash du PBKFD de la phrase secrète réduite.
- `pspK` : phrase de sponsoring cryptée par la clé K du sponsor.
- `YCK` : PBKFD de la phrase de sponsoring cryptée par la clé K du sponsor.
- `hYC` : hash du PBKFD de la phrase de sponsoring,
- `cleAYC` : clé A du sponsor crypté par le PBKFD de la phrase de sponsoring.
- `clePYC` : clé P de la partition (si c'est un compte "O") cryptée par le PBKFD de la phrase de sponsoring.

### `chats`
- `cleCKP` : clé C du chat cryptée,
  - si elle a une longueur inférieure à 256 bytes par la clé K du compte de I.
  - sinon cryptée par la clé RSA publique de I.
- `cleEC` : clé A de l'avatar E cryptée par la clé du chat.

# Périmètre d'un compte

Le périmètre d'un compte délimite un certain nombre de documents:
- un compte n'a la visibilité en session UI que des documents de son périmètre.
- la plupart d'entre eux sont _synchronisés_: une session d'un compte reçoit des _avis de changement_ (pas le contenu) de ces documents qui permettent à l'opération `Sync` de tirer les contenus des documents ayant changé.

### Documents synchronisés du _périmètre_
- **1 espace** : racine et seul document de son sous-arbre, un document`espaces`.
- **1 compte** : ce sous-arbre identifié par l'id du compte comporte trois documents: `comptes comptis invits`.
- **N avatars**: il y un sous-arbre _avatar_ par avatar du compte. Le sous-arbre est identifié par l'id de l'avatar racine et comporte les documents `avatars notes sponsorings chats tickets`
- **N groupes**: il y un sous-arbres _groupe_ par groupe dans lequel un des avatars du compte est actif. Le sous-arbre est identifié par l'id du groupe racine et comporte les documents`groupes notes membres chatgrs`

### Documents NON synchronisés du _périmètre_ 
Ces documents ne sont pas utiles à jour en permanence dans une session. Ils sont lus à la demande en fonction de la phase de dialogue en cours dans la session.
- `syntheses`: id vide (singleton pour l'espace `ns` du compte).
- `partitions`: pour un compte "O", LE document dont l'ID est donnée par `idp` du `comptes`. 
  - Seule une session du Comptable peut accéder, successivement, à tous les documents `partitions` de son espace.
- `comptas` identifié par l'id du compte. 
  - Le Comptable pour tous les comptes, les délégués pour les comptes de leurs partitions, peuvent accéder successivement aux `comptas` des autres comptes.
  - tout compte peut accéder à sa `comptas`.

> Les documents d'un _périmètre_ sont sujet à des évolutions en cours de session suite aux effets des opérations soumises au serveur, 
- soit par la session elle-même, 
- soit par n'importe quelle autre, 
- du même compte ou de n'importe quel autre, 
- marginalement par le GC.
- les changements sont notifiés par PUBSUB aux sessions UI quand ils concernent des documents synchronises du périmètre courant des comptes.

## Disponibilité en session UI
Une session connectée à un compte **dispose en mémoire de tous les documents synchronisés de son compte**:
- chargement initial en début de session,
- puis à réception des avis de changements, rechargement incrémental sélectif des documents ayant changé.

### Avis de changements des `avatars` et `groupes`: document `versions`
Un document `versions` trace une mise à jour, un changement de version d'un document ou plusieurs documents **d'UN** sous-arbre d'un avatar ou d'un groupe:
- (A) un ou plusieurs documents d'UN sous-arbre _avatar_: `avatars` **et ses sous-documents** `notes sponsorings chats tickets`.
  - l'id du document `versions` est l'id du document `avatars` racine du sous-arbre.
- (G) un ou plusieurs documents d'UN sous-arbre _groupe_: `groupes` **et ses sous-documents** `notes membres chatgrs`.
  - l'id du document `versions` est l'id du document `groupes` racine du sous-arbre.

#### Exemple
- mise à jour d'un chat #5 de l'avatar #13;
- le numéro de version associé au sous-arbre A#13 est incrémenté et passe par exemple de 123 à 124;
- le chat #5 prend pour version 124;
- si une session UI était synchronisée pour l'avatar #13 sur la version 112 par exemple, elle va obtenir tous les sous-documents de cet avatar (lui même inclus) de versions supérieure à 112 -qui ont donc changé depuis 112-. Désormais la session sera synchronisée sur la version 126 (la plus récente) pour cet avatar #13.
- elle n'a pas reçu les très nombreux sous-documents ayant une version antérieure à 112 (n'ayant donc pas changé par rapport à l'état connu en mémoire).

## Tracking des créations et mises à jour
**Remarque:** il n'y a pas à proprement parlé de _suppressions_:
- un document `sponsorings` a une date limite de validité: le document est logiquement supprimé dès que cette date est dépassée.
- un document `notes` peut être _vide_, n'a plus de contenu et n'apparaît plus dans les vues, mais son document existe toujours en _zombi_.

Les documents devenus inutiles parce que non référencés par personne sont **purgés**, physiquement détruits.

Les documents `versions` sont chargés du tracking des mises à jour des sous-documents de `avatars` et de `groupes`. Propriétés:
- `id` : ID du groupe ou de l'avatar du document.
- `v` : version, incrémentée de 1 à chaque mise à jour, soit du document maître, soit de ses sous-documents `notes sponsorings chats tickets membres chatgrs`
- `suppr` : jour de _suppression_ de l'avatar ou du groupe (considérés comme _zombi_)

> **Remarque:** Afin d'éviter de conserver pour toujours la trace de très vielles suppressions, le GC lit les `versions` supprimées depuis plus de N mois pour les purger. Les sessions ont toutes eu le temps d'intégrer les disparitions correspondantes.

**La constante `IDBOBS / IDBOBSGC` de `api.mjs`** donne le nombre de jours de validité d'une micro base locale IDB sans resynchronisation. Celle-ci devient **obsolète** (à supprimer avant connexion) `IDBOBS` jours après sa dernière synchronisation. Ceci s'applique à _tous_ les espaces avec la même valeur.

> Les documents `versions` sont purgés par le GC `IDBOBSGC` jours après leur jour de suppression `suppr`.

# Détail des tables / collections _majeures_ et leurs _sous-collections_
Ce sont les documents faisant partie d'un périmètre d'un compte: `partitions comptes comptas comptis invits avatars groupes notes sponsorings chats tickets membres chatgrs versions`

En base de données, les colonnes / propriétés suivantes sont lisibles:
- `id`: l'ID du document précédée du `ns` de l'espace.
- `ids`: pour les sous-documents leur ID secondaire `ids` précédée du `ns` de l'espace.
- `v`: la version du document.
- _quelques_ propriétés devant être indexées, spécifiquement quand elles existent dans la classe du  document:
  - `hk`: la propriété `hk` du document précédée du `ns` de l'espace.
  - `vcv dlv dfh org idf`: valeur de la propriété correspondante du document.
- `_data_`: sérialisation cryptée des propriétés du document.

### `_data_`
Tous les documents, ont en base une propriété `_data_` qui porte toutes les informations sérialisées du document. `_data_` est crypté:
- en base _centrale_ par la clé du site qui a été générée par l'administrateur technique et qu'il conserve en lieu protégé comme quelques autres données sensibles (_token_ d'autorisation d'API, identifiants d'accès aux comptes d'hébergement ...).
- en base _locale_ par la clé K du compte.

Lors de la lecture des documents par l'opération `Sync`, le _data_ d'un document est sérialisé et transmis en retour de la requête POST en HTTPS. Par défaut, toutes les propriétés sont transmises. Toutefois, selon la classe du document et le compte concerné, certaines propriétés _non transmises en session_ sont _omises_ dans la sérialisation du _data_ qui remonte en session.

### `id` et `ids` quand il existe
Ces propriétés sont externalisées et font partie de la clé primaire (en SQL) ou du path (en Firestore).

Pour un `sponsorings` la propriété `ids` est le hash de la phrase de reconnaissance :
- elle est indexée.
- en Firestore l'index est `collection_group` afin de rendre un sponsorings accessible par index sans connaître son _parent_ le sponsor.

### `v` : version d'un document
**La version de 1..n** est incrémentée de 1 à chaque mise à jour,
- soit de son document lui-même: `espaces syntheses partitions comptas`, 
- soit du document `versions` de leurs sous-collections.
  - `avatars notes sponsorings chats tickets`
  - `groupes chatgrs notes membres`

#### `dlv` d'un `transferts`
Elle permet au GC de détecter les transferts en échec et de nettoyer le _storage_.
- en Firestore l'index est `collection_group` afin de s'appliquer aux fichiers des notes de tous les avatars et groupe.

### `dlv` d'un `comptes`
La `dlv` **d'un compte** désigne le dernier jour de validité du compte:
- c'est le **dernier jour d'un mois**.
- **cas particulier**: quand c'est le premier jour d'un mois, la `dlv` réelle est le dernier jour du mois précédent. Dans ce cas elle représente la date de fin de validité fixée par l'administrateur pour l'ensemble des comptes "O". En gros il a un financement des frais d'hébergement pour les comptes de l'organisation jusqu'à cette date (par défaut la fin du siècle).

La `dlv` d'un compte est inscrite dans le document `comptes` du compte: elle est externalisée pour que le GC puisse récupérer tous les comptes obsolètes à détruire.

### `dlv` d'un `sponsorings` 
- jour au-delà duquel le sponsoring n'est plus applicable ni pertinent à conserver. Les sessions suppriment automatiquement à la connexion les sponsorings ayant dépassé leur `dlv`.
- dès dépassement du jour de `dlv`, un sponsorings est purgé (du moins peut l'être).
- elles sont indexées pour que le GC puisse purger les sponsorings. En Firestore l'index est `collection_group` afin de s'appliquer aux sponsorings de tous les avatars.

### `vcv` : version de la carte de visite. `avatars`
Cette propriété est la version `v` du document au moment de la dernière mise à jour de la carte de visite: elle est indexée afin de pouvoir filter un avatar et n'accéder à son contenu que si la version de sa carte de visite est plus récente que celle déjà détenue en session UI.

### `dfh` : date de fin d'hébergement. `groupes`
La **date de fin d'hébergement** sur un groupe permet de détecter le jour où le groupe sera considéré comme disparu. A dépassement de la `dfh` d'un groupe, le GC fait disparaître le groupe inscrivant une `suppr` du jour dans son document `versions`.

### `hk` : hash d'un extrait de la phrase de contact. `avatars`
Cette propriété de `avatars` est indexée de manière à pouvoir accéder à un avatar en connaissant sa phrase de contact. En base la propriété est précédée du `ns` de l'espace.

### `hk` : hash d'un extrait de la phrase secrète. `comptes`
Cette propriété de `comptes` est indexée de manière à pouvoir accéder à un compte en connaissant le `hXR` issu de sa phrase secrète. En base la propriété est précédée du `ns` de l'espace.

# Mémoires caches globale d'une instance de service et de chaque opération 

## Cache locale des `espaces partitions comptes comptis invits comptas avatars groupes versions` dans une instance du service OP

Ces instances ont une mémoire cache des documents _compilés_:
- `comptes` accédés pour vérifier si les listes des avatars et groupes du compte ont changé.
- `comptis` accédés pour avoir les commentaires et hashtags attachés à ses avatars et groupes par un compte.
- `invits` accédé pour avoir les invitations en attente pour un compte.
- `comptas` accédés à chaque changement de volume ou du nombre de notes / chats / participations aux groupes.
- `versions` accédés par l'opération `Sync`.
- `avatars groupes partitions` également fréquemment accédés.

**Les conserver en cache** par leur `id` est une solution naturelle: mais il peut y avoir plusieurs instances s'exécutant en parallèle. 
- Il faut en conséquence interroger la base pour savoir s'il y a une version postérieure et ne pas la charger si ce n'est pas le cas en utilisant un filtrage par `v`. 
- Ce filtrage se faisant sur l'index n'est pas décompté comme une lecture de document quand le document n'a pas été trouvé parce que de version déjà connue.

La mémoire cache est gérée par LRU (tous types de documents confondus) afin de limiter sa taille en mémoire.

## Classe `GD` : Gestionnaire de Documents
Les bases NOSQL (Firestore, DynamoDB, CosmoDB) ont des contraintes vis à vis de la mise à jour transactionnelle (ACID).

Firestore a un concept de transaction marqué par _begin / commit_ **mais** aucune lecture de document ne peut être effectuée après la première écriture dans une transaction. Bref les écritures doivent être exécutées en un _batch_ à la fin de la transaction.

CosmoDB et DynamoDB n'assurent la propriété ACID que dans un _batch_ d'updates et se protègent de mises à jour concurrentes (_update / delete_):
- CosmoDB: en vérifiant que la propriété `Etag` d'un document mis à jour est bien la même que celle du même document avant mise à jour / suppression.
- DynamoDB: en ayant un filtre conditionnel sur la valeur d'une propriété.

`GD` va en conséquence:
- accumuler les _insert / update / delete_ et demander au _provider_ courant le `bulkUpdates` de celles-ci en une fois.
- pour chaque document mis à jour, conserve la propriété `_vav` (version _avant_) du document ce qui est équivalent au `Etga` de CosmoDB et peut servir de filtre pour DynamoDB.

Un objet _gestionnaire de document_ instance de la classe `GD` est en charge de gérer une mémoire cache des documents lus et modifiés au cours d'une opération:
- chaque demande d'un document le lit de la base s'il n'est pas trouvé dans cette cache et l'y inscrit.
- chaque modification d'un document est une modification de l'exemplaire en cache, marqué _mis à jour_.
- à la fin de l'opération,
  - tous les documents créés / mis à jour sont mis à jour en base. La propriété `_vav` (version _avant_) était disponible dans le document lu avant sa mise à jour.
  - les documents _majeurs_ mis à jour sont retournés pour mise à jour à la cache de l'instance de service (avec contrôle de la croissance de leur version).

Ce protocole n'est pas contraignant, plutôt une facilité, par rapport à un usage classique, typiquement avec une base relationnelle.

# Clés et identifiants

## Le hash SHA256
Son résultat fait 32 bytes. Il est rapide à calculer. Il est utilisé par _hash court_ (voir ci-dessous).

## Le hash PBKFD
Son résultat fait 32 bytes. Long à calculer, son algorithme ne le rend pas susceptible d'être accéléré par usage de CPU graphiques. Il est considéré comme incassable par force brute.

## Les clés AES
Ce sont des bytes de longueur 32. Un texte crypté a une longueur variable :
- quand le cryptage est spécifié _libre_ le premier byte du texte crypté est le numéro du _salt_ choisi au hasard dans une liste pré-compilée : un texte donné 'AAA' ne donnera donc pas le même texte crypté à chaque fois ce qui empêche de pouvoir tester l'égalité de deux textes cryptés au vu de leurs valeurs cryptées.
- quand le cryptage est _fixe_ le numéro de _salt_ est 1 : l'égalité de valeurs cryptées traduit l'égalité de leur valeurs sources.

## Les clés RSA
La clé de cryptage (publique) et celle de décryptage (privée) sont de longueurs différentes. 

Le résultat d'un cryptage a une longueur fixe de 256 bytes. Deux cryptages RSA avec la même clé d'un même texte donnent deux valeurs cryptées différentes.

Le cryptage / décryptage est long et le texte à crypter doit avoir une longueur maximale de 256 bytes.

# Hash _court_
Le hash _court_ d'un bytes (ou d'un string) est un hash SHA256, replié sur 9 bytes et encodé en base64 où les caractères `+` et `/` sont replacés par `0` et `1` (les signes `=` sont supprimés).

Le résultat est un string de 12 signes `0-9 a-z A-Z`.

## Un entier sur 53 bits est intègre en Javascript
Le maximum 9,007,199,254,740,991 fait 16 chiffres décimaux si le premier n'est pas 9. Il peut être issu de 6 bytes aléatoires.

## Dates et date-heures
Les date-heures sont exprimées en millisecondes depuis le 1/1/1970, un entier intègre en Javascript (ce serait d'ailleurs aussi le cas pour une date-heure en micro-seconde).

Les dates sont exprimées en `aaaammjj` sur un entier (géré par la class `AMJ`). Ce sont des dates UTC, mais elles peuvent s'afficher en date _locale_.

## Clés des documents
Les clés sont des 32 bytes aléatoires dont le premier byte est surchargé à:
- `1` pour une clé d'espace,
- `2` pour une clé de partition,
- `3` pour une clé d'avatar,
- `4` pour une clé de groupe.

**Exception pour le Comptable:** sa clé est formée d'un byte à 3 et de 31 bytes à 0.

## IDs des documents
Les ID des documents ci-dessus sont calculés ainsi:
- un hash court de leur clé est calculé (donc 12 signes, le base64 d'un hash de 9 bytes).
- le premier caractère est remplacé par `1` à `4` selon la classe de document.

**Exception pour le Comptable:** son ID est `300000000000`.

**Remarque**: en présence d'un id on sait donc la classe du document correspond (donnée par son premier caractère). En affichage des IDs, les 4 derniers signes sont utilisés.

# Authentification

## L'administrateur technique
Il a une phrase de connexion dont le SHA256 de son PBKFD (`shax`) est enregistré dans la configuration d'installation. 
- Il n'a pas d'id, ce n'est PAS un compte.
- Une opération de l'administrateur est repérée parce que son _token_ contient son `shax`.

**Quelques opérations ne sont pas authentifiées**: 
- L'opération de création d'un compte `AccepterSponsorings`: par principe le compte n'est pas encore enregistré.
- Les opérations du GC,
- des opérations de nature _ping_ tests d'écho, tests d'erreur fonctionnelle.

## `sessionId`: `pageId.nc`
`pageId` est générée au hasard (c'est un hash _court_ sur 12 lettres / chiffres) au chargement de l'application.

A chaque nouvelle connexion à un compte, un numéro de connexion `nc` est incrémenté de 1. 

L'identifiant d'une connexion est `sessionId` : `pageId.nc`.

## Token d'authentification
Toute opération ayant à authentifier son émetteur porte un `token` sérialisation encodée en base 64 URL de `{ sessionId, org, shax, hXR, hXC }`:
- Pour l'administrateur technique:
  - `org`: `admin`
  - `shax` : SHA256 du PBKFD de sa phrase secrète en base64.
- Pour un compte connecté:
  - `org` : le code l'organisation.
  - `sessionId`,
  - `hXR` : hash (sur 14 chiffres) du PBKFD d'un extrait de la phrase secrète.
  - `hXC` : hash (sur 14 chiffres) du PBKFD de la phrase secrète complète.

Le service OP recherche le document `comptes` par `ns + hXR` (propriété `hk` indexée de `comptes`). Le `ns` est connu par le code `org` figurant dans le token.
- vérifie que `hXC` est bien celui enregistré dans `comptes`.
- enregistre dans le contexte de l'opération `sessionId, org, ns`.

# _Textes_ humainement interprétables

**Les photos des cartes de visites sont assimilées par la suite par simplification à des _textes_.**

Ces textes humainement interprétables sont toujours cryptés par des clés qu'un compte obtient,
- indirectement par sa clé K cryptée par sa phrase secrète,
- par des _contacts_ qui lui ont communiqué leur clé de carte de visite (contacts directs ou membre d'un groupe).
- par la clé d'un groupe dont la clé a été transmise lors de l'invitation.

On les trouvent en propriétés:
- `ph`: **photo d'une carte de visite** encodée en base64 URL.
- `tx`: **texte d'une carte de visite**: le début de la _première ligne_ donne un _nom_, le reste est un complément d'information.
- `tx`: **texte d'un échange** sur un `chats` ou `chatgrs`.
- `tx`: **commentaire personnel d'un compte** attaché à un avatar contact (chat ou membre d'un groupe) ou à un groupe.
- `ht` : **hashtags**, suite de mots attachés par un compte à,
  - _un avatar_ (chat ou membre d'un groupe),
  - _un groupe_ dont il est ou a été membre ou au moins invité,
  - _une note_, personnelle ou d'un des groupes où il est actif et a accès aux notes.
- `texte` d'une note personnelle ou d'un groupe.

Les `texte / tx` sont gzippés ou non avant cryptage: c'est automatique dès que le texte a une certaine longueur.

> **Remarque:** Les services OP et PUBSUB ne voient **jamais en clair**, aucun texte, ni aucune clé susceptible de crypter un texte, ni la clé K des comptes, ni les _phrase secrètes_ ou _phrases de contacts / sponsorings_.

> Les textes sont cryptés / décryptés par l'application Web. Si celle-ci est malicieuse / boguée, les textes sont illisibles mais finalement pas plus que ceux qu'un utilisateur qui les écrirait en idéogrammes pour un public occidental ou qui inscrirait des textes absurdes.

# Sous-objet `Notification`

Un objet _notification_ est immuable, en cas de _mise à jour_ il est remplacé par un nouveau.

Type des notifications:
- E : _de l'espace_. Elle concerne tous les comptes et est déclarée par l'administrateur du site.
- P : _d'une partition_. Elles concernent chacune tous les comptes "O" **d'une partition**. Elles sont déclarées, soit par le Comptable, soit par un de ses _délégués_ sur cette partition.
- C : _d'un compte_. Elles concernent chacune **un seul compte "O"**.

Une notification a les propriétés suivantes:
- `nr`: restriction d'accès: 
  - 1 : **aucune restriction**. La notification est informative (le texte peut annoncer une restriction imminente).
  - 2 : **restriction réduite**
    - E : espace figé
    - P : accès en lecture seule
    - C : accès en lecture seule
  - 3 : **restriction forte**
    - E : espace clos
    - P : accès minimal
    - C : accès minimal
- `dh` : date-heure de création.
- `texte`: il porte l'information explicative.
  - type E: en clair.
  - types P et C: crypté par la clé P de la partition.
- `idDel`: id du délégué ayant créé cette notification pour un type P ou C quand ce n'est pas le Comptable.

**Remarque:** une notification `{ dh: ... }` correspond à la suppression de la notification antérieure (ni restriction, ni texte).

> Le document `comptes` a une date-heure de lecture `dhvuK` qui indique _quand_ le titulaire du compte a lu les notifications. Une icône peut ainsi signaler l'existence d'une _nouvelle_ notification, i.e. une notification qui n'a pas été lue.

# Sous-objet `CV` carte de visite

Une carte de visite a 4 propriétés `{ id, v, ph, tx }`:
- `id` : de l'avatar ou du groupe.
- `v`: version de la carte de visite, version du groupe ou de l'avatar au moment de sa dernière mise à jour.
- `ph`: photo cryptée par la clé A de l'avatar ou G du groupe propriétaire.
- `tx`: texte (gzippé) crypté par la clé A de l'avatar ou G du groupe propriétaire.

`nom` : il correspond aux 16 premiers caractères de la première ligne du texte. Ce nom est affiché partout ou l'avatar / groupe apparaît, suivi des 4 derniers chiffres de son id.

Les cartes de visite des avatars sont hébergées dans le document `avatars`, celles des groupes dans leurs documents `groupes`.

Les cartes de visites des avatars sont dédoublées dans d'autres documents:
- `membres` : chaque membre y dispose de sa carte de visite.
- `chats` : chaque interlocuteur dispose de la carte de visite de l'autre.

## Mises à jour des cartes de visite des membres
- la première inscription se fait à l'ajout de l'avatar comme _contact_ du groupe.
- le rafraîchissement peut être demandé pour un groupe donné.
  - pour chaque membre, l'opération compare la version détenue dans le membre et la version détenue dans l'avatar. Cette vérification ne fait intervenir que des filtres sur les index si la version dans `membres` est à jour.
  - si la version de `membres` n'est pas à jour, elle est mise à jour. 
- en session, lorsque la page listant les membres d'un groupe est ouverte, elle peut envoyer une requête de rafraîchissement des cartes de visite.

### Mise à jour dans les chats
- à la mise à jour d'un chat, les cartes de visite des deux côtés sont rafraîchies si nécessaire.
- le rafraîchissement peut être demandé pour tous les chats d'un avatar donné.
  - pour chaque chat, l'opération compare la version détenue dans le chat et la version détenue dans l'avatar. Cette vérification ne fait intervenir que des filtres sur les index si la version dans chat est à jour.
  - si la version dans chat n'est pas à jour, elle est mise à jour. 
- en session, lorsque la page listant les chats d'un avatar est ouverte, elle peut envoyer une requête de rafraîchissement des cartes de visite.

# Documents `versions`

Un document `versions` donne la plus haute version d'un sous-arbre:
- avatar: `avatars notes sponsorings chats tickets`,
- groupe: `groupes notes membres`.

_data_ :
- `id` : ID du document.
- `v` : 1..N, plus haute version attribuée aux documents du sous-arbre.
- `suppr` : jour de suppression, ou 0 s'il est actif.

Quand un document, un `chats` par exemple est mis à jour, l'opération,
- lit le document `versions` de son sous-arbre:
- lit le document de l'avatar (racine du sous-arbre) de même `id` que le `chats`,
- incrémente de 1 de `v` de `versions`,
- inscrit v comme version `v` de `chats`,
- met à jour de `versions` et `chats`.

La version `v` est celle de tout le sous-arbre, la plus haute attribuée à un document du sous-arbre.

## Documents `espaces`

Ces documents sont créés par l'administrateur technique à l'occasion de la création de l'espace et du Comptable correspondant.

**Il est _synchronisé_ en session UI:** un avis de changement (avec la nouvelle valeur de sa version) est poussé par le service PUBSUB à toutes les sessions en cours du même espace (ns).

La lecture effective du document vérifie l'habilitation à sa lecture et ne transmet que les propriétés autorisées.
- la propriété `ns` est récupérée par le service OP, elle n'est pas stockée dans le document `espaces`.
- _Administrateur technique_ : toutes les propriétés, quelque soit l'espace.
- _Comptable_ : toutes les propriétés (pour _son_ espace, il ne peut pas lire les autres.
- _Délégués_ : pour leur espace seulement, toutes les propriétés sauf `moisStat moisStatT dlvat nbmi`
- _autres comptes_: restriction d'un délégué, mais de plus la seule notification de _leur_ partition (s'il y en a une) est transmise.

**Les sessions sont systématiquement synchronisées à _leur_ espace.** Elles sont ainsi informées à tout instant d'un changement des notifications,
- E de l'espace lui-même,
- de leur partition (pour un compte "O").

> **Remarque**: les notifications C (de compte) sont portées par les documents `partitions` **et** `comptes` et sont synchronisées par lui. C'est aussi le cas des dépassements de seuils (`pcn pcv` pour les quotas, `pcc nbj` pour la consommation) qui remontent de `comptas` à `comptes` lors de franchissement de variation significative -5% ou 5 jours- (pas à chaque opération).

_data_ :
- `id` : string vide.
- `v` : 1..N
- `org` : code de l'organisation propriétaire.

- `ns` : **cette propriété N'EST PAS persistante**. Elle calculée par le service OP et transmise en session UI.

- `creation` : date de création.
- `moisStat` : dernier mois de calcul de la statistique des comptas.
- `moisStatT` : dernier mois de calcul de la statistique des tickets.
- `quotas`:  `{ qn, qv, qc }` : quotas maximum globaux attribués par l'administrateur technique.
- `dlvat` : `dlv` déclarée par l'administrateur technique.
- `cleES` : clé de l'espace cryptée par la clé du site. Permet au comptable de lire les reports créés sur le serveur et cryptés par cette clé E.
- `notifE` : notification pour l'espace de l'administrateur technique. Le texte n'est pas crypté.
- `opt`: option des comptes autonomes.
  - 0: Pas de comptes "autonomes",
  - 1: Comptes autonomes autorisés.
- `nbmi`: nombre de mois d'inactivité acceptable pour un compte "O" fixé par le comptable. Ce changement n'a pas d'effet rétroactif.
- `tnotifP` : map des notifications de niveau _partition_.
  - _clé_ : ID de la partition.
  - _valeur_ : notification (ou `null`), texte crypté par la clé P de la partition.

_Remarques:_
- `opt nbmi` : sont mis à jour par le Comptable. `opt`:
- `tnotifP` : mise à jour par le Comptable et les délégués des partitions.

**Au début de chaque opération, l'espace est lu afin de vérifier la présence de notifications E et P** (éventuellement restrictives) de l'espace et de leur partition (pour un compte "O"):
- c'est une lecture _lazy_ : si l'espace a été trouvé en cache et relu depuis la base depuis moins de 5 minutes, on l'estime à jour.
- en conséquence, _quand il y a plusieurs instances en parallèle_, la prise en compte de ces notifications n'est _certaine_ qu'au bout de 5 minutes.

### `dlvat nbmi`
L'administrateur technique gère une `dlvat` pour l'espace : 
- c'est la date à laquelle l'administrateur technique détruira les comptes. Par défaut elle est fixée à la fin du siècle.
- l'administrateur ne peut pas (re)positionner une `dlvat` à moins de `nbmi` mois du jour courant afin d'éviter les catastrophes de comptes supprimés sans que leurs titulaires n'aient eu le temps de se reconnecter.

L'opération de mise à jour d'une `dlvat` est une opération longue du fait du repositionnement des `dlv` des comptes égales à la `dlvat` remplacée. C'est une **tâche**.

**Le maintien en vie d'un compte en l'absence de connexion** a le double inconvénient, 
- d'immobiliser des ressources peut-être pour rien,
- d'augmenter les coûts d'avance sur les frais d'hébergement.

Le Comptable fixe en conséquence un `nbmi` (de 3, 6, 12, 18, 24 mois),
- évitant de contraindre les comptes à des connexions fréquentes rien que pour maintenir le compte en vie, 
- évitant que les comptes oublient de le faire et se voient automatiquement résiliés après un délai trop bref de non utilisation de leur compte.
- l'usage d'un `nbmi` à 3 mois se justifie par exemple pour un site de démonstration où les comptes sont fictifs et s'auto-détruisent rapidement.

> Il n'y a aucun moyen dans l'application pour contacter le titulaire d'un compte dans la _vraie_ vie, aucun identifiant de mail / téléphone, etc.

## Protocole de création d'un espace et de son Comptable
**Par l'Administrateur Technique**: création d'un espace:
- choix du code de l'espace `ns` et de l'organisation `org`
- acquisition de la phrase de sponsoring du comptable T -> `TC` (son PBKFD) -> `hTC` (son hash)
- **Opération** `CreationEspace`
  - Arguments: `ns org TC hTC`
  - Traitement:
    - OK si: 
      - soit espace n'existe pas, 
      - soit espace existe et a un `hTC` : re-création avec une nouvelle phrase de sponsoring.
    - génération de la `cleE` de l'espace: -> `cleET` (par TC) et `cleES` (par clé système).
    - stocke dans l'espace: `hTC cleES cleET`. Il est _à demi_ créé, son Comptable n'a pas encore créer son compte.

**Par le Comptable**: création de son compte
- saisie de la phrase de sponsoring T -> `hTC TC`
- **Opération** `GetCleET`:
  - argument: `org hTC` (pour vérification)
  - retour: `cleET`
- saisie phrase secrète du compte: X -> `XC` -> `hXR hXC`
- génération de la clé K: -> `cleKXC` -> `cleEK`
- génération pub/priv: -> `privK pub`
- génération de la clé P de la partition 1: `clePK` -> `ck` `{cleP, code}` crypté par clé K
- **Opération** `CreationComptable`:
  - arguments: `org hTC` (pour vérification) `hXR hXC cleK clePK privK pub cleAP cleAK cleKXC clePA ck`
    - implicite: `id` du Comptable, génération `rds` du compte et de son avatar principal 
  - Traitement:
    - création de `compte compti compta` du Comptable
    - création de la `partition` 1 ne comprenant que le Comptable
    - création de son `avatar` principal (et pour toujours unique)
    - _dans son `espace`_: suppression de `hTC`

# Document `syntheses` d'un espace

Ces documents sont des singletons de leur espace. Ils ne sont pas synchronisés, les sessions UI les demandent explicitement,
- pour l'administrateur technique,
- pour le Comptable.

_data_:
- `id` : string vide.
- `v` : version, numéro d'ordre de mise à jour.
- `qA` : `{ qc, qn, qv }` - quotas **maximum** disponibles pour les comptes A.
- `qtA` : `{ qc, qn, qv }` - quotas **effectivement attribués** aux comptes A. En conséquence `qA.qn - qtA.qn` est le quotas `qn` encore attribuable aux compte A.

- `dh` : date-heure de dernière mise à jour (à titre informatif).
- `tsp` : map des _synthèses_ des partitions.
  - _clé_: id de la partition.
  - _valeur_ : `synth`, objet des compteurs de synthèse calculés de la partition.
    - `id nbc nbd`
    - `ntfp[1,2,3]`
    - `q` : `{ qc, qn, qv }`
    - `qt` : `{ qc qn qv c2m n v }`
    - `ntf[1,2,3]`
    - `pcac pcan pcav pcc pcn pcv`

Une agrégation des `synth[i]` est compilé en `tsp['0']`.
- `q` : `{qc, qn, qv}` sont les quotas totaux maximum pour l'ensemble des partitions.

Le document `syntheses` est mis à jour à chaque fois qu'un document `partitions` l'est: le `synth` de la partition est reporté dans l'élément correspondant de `tsp`. En cas de suppression d'une partition son entrée est supprimée.

# Documents `partitions` des partitions d'un espace

Une partition est créée par le Comptable qui peut la supprimer quand il n'y a plus de comptes attachés à elle. L'identifiant d'une partition est aléatoire attribué par le Comptable à sa création.

**La clé P d'une partition** sert uniquement à crypter les textes des notifications de niveau _P partition_ ou C relatif à un compte.
- elle est générée à la création de la partition,
- elle est transmise aux comptes rattachés qui la détiennent dans la propriété `clePK`,
  - soit à leur création par sponsoring : elle est cryptée par la clé K du compte créé.
  - soit quand le compte change de partition (par le Comptable) ou passe de compte "A" à compte "O" par un délégué ou le Comptable: elle est cryptée par la clé publique RSA du compte.

**Un document partition NON synchronisé, est explicitement demandé** par les sessions UI,
- soit du Comptable,
- soit d'un délégué.
- soit d'un compte "O" non délégué. Dans ce cas:  
  - dans la map `mcpt`, seules les entrées des délégués sont présentes.
  - les compteurs de quotas / consommation d'un délégué sont à 0.
  - la `cleAP` est disponible ce qui permet de contacter les _délégués_ pour un _chat d'urgence_.

**Principales opérations**
- attachement / détachement d'un compte à une partition.
- attribution / retrait de son statut de délégué.
- pose / retrait d'une notification de niveau P ou C (pour un seul compte). La notification C est redondée dans le compte.
- modification des quotas globaux de la partition.
- modification des quotas attribués à un compte.

#### Incorporation des consommations des comptes
Les compteurs de consommation d'un compte extraits de `comptas` sont recopiés à l'occasion de la fin d'une opération:
- dans les compteurs `{ qc, qn, qv, pcc, pcn, pcv, nbj }` du document `comptes`,
- pour un compte "O" dans les compteurs `q: { qc qn qv c2m nn nc ng v }` de l'entrée du compte dans son document `partitions`.
  - en conséquence la ligne de synthèse de sa partition est reportée dans l'élément correspondant de son document `syntheses`.
- afin d'éviter des mises à jour trop fréquentes, la procédure de report n'est engagée qui si les compteurs `pcc pcn pcv` passe un cap de 5% ou que `nbj` passe un cap de 5 jours.

> **Remarque**: la modification d'un compteur de quotas `qc qn qv` provoque cette procédure de report `comptas -> comptes -> partitions -> syntheses` à chaque évolution et sans effet de seuil. 

_data_:
- `id` : ID de la partition attribué par le Comptable à sa création.
- `v` : 1..N

- `nrp`: niveau de restriction de la notification (éventuelle) de niveau _partition_ mémorisée dans `espaces` et dont le texte est crypté par la clé P de la partition.
- `q`: `{ qc, qn, qv }` quotas globaux attribués à la partition par le Comptable.
- `mcpt` : map des comptes "O" attachés à la partition. 
  - _clé_: id du compte.
  - _valeur_: `{ notif, cleAP, del, q }`
    - `notif`: notification du compte cryptée par la clé P de la partition (redonde celle dans compte).
    - `cleAP` : clé A du compte crypté par la clé P de la partition.
    - `del`: `true` si c'est un délégué.
    - `q` : `qc qn qv c2m nn nc ng v` extraits du document `comptas` du compte.
      - `c2m` est le compteur `conso2M` de compteurs, montant moyen _mensualisé_ de consommation de calcul observé sur M/M-1. 

`mcpt` compilé en session - Ajout à `q` :
  - `pcc` : pourcentage d'utilisation de la consommation journalière `c2m / qc`
  - `pcn` : pourcentage d'utilisation effective de qn : `nn + nc ng / qn`
  - `pcv` : pourcentage d'utilisation effective de qc : `v / qv`

**Un objet `synth` est calculable** (en session ou dans le serveur):
- `qt` : les totaux des compteurs `q` : (`qc qn qv c2m n (nn+nc+ng) v`) de tous les comptes,
- `ntf`: [1, 2, 3] - le nombre de comptes ayant des notifications de niveau de restriction 1 / 2 / 3. 
- `nbc nbd` : le nombre total de comptes et le nombre de délégués.
- _recopiés de la racine dans `synth`_ : `id nrp q`
- plus, calculés localement :
  - `pcac` : pourcentage d'affectation des quotas : `qt.qc / q.qc`
  - `pcan` : pourcentage d'affectation des quotas : `qt.qn / q.qn`
  - `pcav` : pourcentage d'affectation des quotas : `qt.qv / q.qv`
  - `pcc` : pourcentage d'utilisation de la consommation journalière `qt`.`c2m / q.qc`
  - `pcn` : pourcentage d'utilisation effective de `qn` : `qt.n / q.qn`
  - `pcv` : pourcentage d'utilisation effective de `qc` : `qt.v / q.qv`

## Documents `comptes`

Un document `comptes` est identifié par l'id du compte: il est **synchronisé en session par son `rds`** et y est toujours disponible à jour. Sa _lecture_ ne se fait que par l'opération `Sync`.

_data_ :
- `id` : ID du compte = ID de son avatar principal.
- `v` : 1..N.
- `hk` : `hXR`, hash du PBKFD d'un extrait de la phrase secrète (en base précédé de `ns`).
- `dlv` : dernier jour de validité du compte.

- `dharF dhopf dharC dhopC dharS dhopS`: dh des ACCES RESTREINT (F, C, S).

- `vpe` : version du périmètre
- `vci` : version de `comptis`
- `vin` : version de `invits`

- `hXC`: hash du PBKFD de la phrase secrète complète (sans son `ns`).
- `cleKXC` : clé K cryptée par XC (PBKFD de la phrase secrète complète).
- `cleEK` : pour le Comptable, clé de l'espace cryptée par sa clé K à la création de l'espace pour le Comptable. Permet au comptable de lire les reports créés sur le serveur et cryptés par cette clé E.
- `privK` : clé privée RSA de son avatar principal cryptée par la clé K du compte.

- `dhvuK` : date-heure de dernière vue des notifications par le titulaire du compte, cryptée par la clé K.
- `qv` : `{ qc, qn, qv, nn, nc, ng, v, cjm }`. Recopiés de `compta.compteurs.qv` en fin d'opération quand l'un d'eux passe un seuil de 5% (sans seuil pour `qc, qn, qv`), à la montée ou à la descente.
  - `qc`: limite de consommation
  - `qn`: quota du nombre total de notes / chats / groupes.
  - `qv`: quota du volume des fichiers.
  - `nn`: nombre de notes existantes.
  - `nc`: nombre de chats existants.
  - `ng` : nombre de participations aux groupes existantes.
  - `v`: volume effectif total des fichiers
  - `cjm`: coût journalier moyen de calcul sur M et M-1.
- `flags` : flags courants. Recopiés de `compta.compteurs.flags` en fin d'opération en cas de changement.
  - `RAL` : ralentissement (excès de calcul / quota)
  - `NRED` : documents en réduction (excès de nombre de documents / quota)
  - `VRED` : volume de fichiers en réduction (excès de volume / quota)
  - `ARSN` : accès restreint pour solde négatif

- `lmut` : liste des `ids` des chats pour lesquels le compte (son avatar principal) a une demande de mutation (`mutI` != 0)

_Comptes "O" seulement:_
- `clePK` : clé P de la partition cryptée par la clé K du compte. Si cette clé a une longueur de 256, la clé P a été cryptée par la clé publique de l'avatar principal du compte suite à une affectation à une partition APRÈS sa création (changement de partition, passage de compte A à O)
- `idp` : ID de sa partition.
- `del` : `true` si le compte est délégué de la partition.
- `notif`: notification de niveau _compte_ dont le texte est crypté par la clé P de la partition (`null` s'il n'y en a pas).

- `mav` : map des avatars du compte. 
  - _clé_ : ID de l'avatar.
  - _valeur_ : `claAK`: clé A de l'avatar crypté par la clé K du compte.

- `mpg` : map des participations aux groupes:
  - _clé_ : ID du groupe
  - _valeur_: `{ cleGK, lav }`
    - `cleGK` : clé G du groupe cryptée par la clé K du compte.
    - `lav`: liste de ses avatars participant au groupe.

**Comptable seulement:**
- `tpK` : map des partitions cryptée par la clé K du Comptable `[ {cleP, code }]`. Son index est le numéro de la partition.
  - `cleP` : clé P de la partition.
  - `code` : code / commentaire court de convenance attribué par le Comptable.

#### `vci vin` : synchronisation des `comptis invits`
A chaque mise à jour du `comptis` (resp. `invits`) du compte, la version courante du compte est inscrite, 
- comme `v` de `comptis` (resp. de invits), 
- dans `vci` (resp. `vin`). 

Ceci permet lors de l'appel `Sync` en session UI à la réception d'un avis de changement du compte, de savoir si `comptis` (resp. `invits`) est hors date ou non et doit ou non être rechargé. 

#### Périmètre du compte, `vpe`
Le périmètre d'un compte est la lis ordonnée sans doublon (un Set ordonné) des IDs des avatars trouvés dans `mav` et des groupes trouvés dans `mpg`.

A la fin de chaque opération, le service OP compare, pour tous les comptes mis à jour par l'opération, les périmètres avant et après l'opération: la liste des périmètres changés `[[ID du compte, nouveau périmètre [id1 ... ]] ... ]` est transmise à PUBSUB afin d're mis à jour et de pouvoir notifier les futurs changements.

Si le périmètre d'un compte a changé, la propriété vpe est mise à jour dans le compte avec la valeur courante de la version du compte. Ceci permet de savoir quand un compte a changé, si son périmètre a changé ou non depuis la version précédente et à PUBSUB de mettre à jour ou non le périmètre d'un compte.

# Documents `comptis`

Ils sont identifiés pat l'ID de leur compte, créé et purgé avec lui. C'est une prolongation du document `comptes` portant des informations personnelles (texte et hashtags) à propos des avatars et groupes connus du compte.

**Ils sont synchronisés:** la _lecture_ en session ne s'effectue que par l'opération `Sync`.

_data_:
- `id` : id du compte.
- `v` : version.

- `mc` : map des contacts (des avatars) et des groupes _connus_ du compte,
  - _cle_: `id` de l'avatar ou du groupe,
  - _valeur_ : `{ ht, tx }`. Hashtags et texte attribués par le compte.
    - `ht` : suite des hashtags séparés par un espace et cryptée par la clé K du compte.
    - `tx` : commentaire gzippé et crypté par la clé K du compte.

# Documents `invits`

Ils sont identifiés pat l'ID de leur compte, créé et purgé avec lui. C'est une prolongation du document `comptes` portant la liste des invitations à des groupes adressées à un des avatars du compte.

**Ils sont synchronisés:** la _lecture_ en session ne s'effectue que par l'opération `Sync`.

_data_:
- `id` : id du compte.
- `v` : version.

- `invits`: liste des invitations en cours:
  - _valeur_: `{idg, ida, cleGA, cvG, ivpar, dh}`
    - `idg`: id du groupe,
    - `ida`: id de l'avatar invité
    - `cleGA`: clé du groupe crypté par la clé A de l'avatar.
    - `cvG` : carte de visite du groupe (photo et texte sont cryptés par la clé G du groupe).
    - `flags` : d'invitation.
    - `invpar` : `[{ cleAG, cvA }]`
      - `cleAG`: clé A de l'avatar invitant crypté par la clé G du groupe.
      - `cvA` : carte de visite de l'invitant (photo et texte sont cryptés par la clé G du groupe). 
    - `msgG` : message de bienvenue / invitation émis par l'invitant.

Pour un simple contact:
- `flags` est à 0
- `msgG` est null
- `invpar` reflète dans le cas des invitations unanimes, la liste des votants _pour_ à cet instant.

Un _contact_ peut se faire effacer des contacts du groupe et s'inscrire en liste noire.

# Documents `comptas`

**Ces documents de même id que leur compte est lu à chaque début d'opération et mis à jour par l'opération.**
- si ses compteurs `pcc, pcn, pcv, nbj` _ont changé d'ordre de grandeur_ (5% / 5j) ils sont reportés dans le document `comptes`: ce dernier ne devrait, statistiquement, n'être mis à jour que rarement en fin d'opération.

**Les documents ne sont PAS synchronisés.** La lecture est à la demande par les sessions, ce qui permet de vérifier qui le demande: compte lui-même, Comptable, un délégué de sa partition pour un compte "O".

_data_:
- `id` : numéro du compte = id de son avatar principal.
- `v` : 1..N. Sa version lui est spécifique.

- `qv` : `{qc, qn, qv, nn, nc, ng, v}`: quotas et nombre de groupes, chats, notes, volume fichiers. Valeurs courantes.
- `estA` : `true` si c'est un compte A.
- `compteurs` sérialisation des quotas, volumes et coûts.
- _Comptes "A" seulement_
  - `solde`: résultat, 
    - du cumul des crédits reçus depuis le début de la vie du compte (ou de son dernier passage en compte A), 
    - plus les dons reçus des autres,
    - moins les dons faits aux autres.
  - `tickets`: map des tickets / dons:
    - _clé_: `ids` du ticket,
    - _valeur_: `{dg, dr, ma, mc, refa, refc }`
  - `dons` : liste des dons effectués / reçus `[{ dh, m, iddb }]`
    - `dh`: date-heure du don
    - `m`: montant du don (positif ou négatif)
    - `iddb`: id du donateur / bénéficiaire (selon le signe de `m`).

# Documents `avatars`

Un compte a un avatar principal de même ID que lui et peut avoir des avatars secondaires ayant chacun leur propre ID.

_data_:
- `id` : ID de l'avatar.
- `v` : 1..N.
- `vcv` : version de la carte de visite afin qu'une opération puisse détecter (sans lire le document) si la carte de visite est plus récente que celle qu'il connaît.
- `hk` : `hZR` hash du PBKFD de la phrase de contact réduite (précédé du `ns` en base).
- `alias` : ID alternative générée à la création de l'avatar. Identifie l'avatar sur Storage pour éviter d'y faire figurer son ID réelle.

- `idc` : id du compte de l'avatar (égal à son id pour l'avatar principal).
- `cleAZC` : clé A cryptée par ZC (PBKFD de la phrase de contact complète).
- `pcK` : phrase de contact complète cryptée par la clé K du compte.
- `hZC` : hash du PBKFD de la phrase de contact complète.

- `cvA` : carte de visite de l'avatar `{id, v, ph, tx}`. photo et texte (possiblement gzippé) cryptés par la clé A de l'avatar.

- `pub privK` : couple des clés publique / privée RSA de l'avatar.

# Documents `tickets`

Ce sont des sous-documents de `avatars` qui n'existent **que** pour l'avatar principal du Comptable.

Il y a un document `tickets` par ticket de crédit généré par un compte "A" annonçant l'arrivée d'un paiement correspondant. Chaque ticket est dédoublé:
- un exemplaire dans la sous-collection `tickets` du Comptable,
- un exemplaire dans le document `comptas` du compte, dans la liste `tickets` cryptée par la clé K du compte A `{ids, dg, dr, ma, mc, refa, refc, di }`.

_data_:
- `id`: ID du Comptable.
- `ids` : identifiant du ticket
- `v` : version du ticket.

- `dg` : date de génération.
- `dr`: date de réception. Si 0 le ticket est _en attente_.
- `ma`: montant déclaré émis par le compte A.
- `mc` : montant déclaré reçu par le Comptable.
- `refa` : code court (32c) facultatif du compte A à l'émission.
- `refc` : code court (32c) facultatif du Comptable à la réception.
- `disp`: true si le compte était disparu lors de la réception.
- `idc`: id du compte générateur. Cette donnée n'est pas transmise aux sessions.

## Cycle de vie
#### Génération d'un ticket (annonce de paiement) par le compte A
- le compte A déclare,
  - un montant `ma` celui qu'il affirme avoir payé / viré.
  - une référence `refa` textuelle libre facultative à un dossier de _litige_, typiquement un _avoir_ correspondant à une erreur d'enregistrement antérieure.
- le ticket est généré et enregistré en deux exemplaires:
  - un dans le document `comptas` du compte,
  - un comme document `tickets` du Comptable..

#### Effacement d'un de ses tickets par le compte A
En cas d'erreur, un ticket peut être effacé par son émetteur, _à condition_ d'être toujours _en attente_ (ne pas avoir de date de réception). Le ticket est physiquement effacé de `tickets` et de la liste `comptas.tickets`.

#### Réception d'un paiement par le Comptable
- le Comptable ne peut _que_ compléter un ticket _en attente_ (pas de date de réception) **dans le mois d'émission du ticket ou les deux précédents**. Au delà le ticket est _auto-détruit_.
- sur le ticket correspondant le Comptable peut remplir:
  - le montant `mc` du paiement reçu, sauf indication contraire par défaut égal au montant `ma`.
  - une référence textuelle libre `refc` justifiant une différence entre `ma` et `mc`. Ce peut être un numéro de dossier de _litige_ qui pourra être repris ensuite entre le compte A et le Comptable.
- la date de réception `dr` est inscrite, le ticket est _réceptionné_.
- le ticket est mis à jour dans `tickets` et dans la liste `comptas.tickets` du compte A: **le compte A est crédité**.

#### Lorsque le compte A va sur sa page de gestion de ses crédits
- les tickets dont il possède une version plus ancienne que celle détenue dans `tickets` du Comptable sont mis à jour.
- les tickets émis un mois M toujours non réceptionnés avant la fin de M+2 sont supprimés.
- les tickets de plus de 2 ans sont supprimés.

**Remarques:**
- de facto dans `tickets` un document ne peut avoir qu'au plus deux versions.
- la version de création qui créé le ticket et lui donne son identifiant secondaire et inscrit les propriétés `ma` et éventuellement `refa` désormais immuables.
- la version de réception par le Comptable qui inscrit les propriétés `dr mc` et éventuellement `refc`. Le ticket devient immuable dans `tickets`.
- les propriétés sont toutes immuables.

#### Listes disponibles en session
Un compte "A" dispose de la liste de ses tickets sur une période de 2 ans, quelque soit leur statut, y compris ceux obsolètes parce que non réceptionnés avant fin M+2 de leur génération.

Le Comptable dispose en session de la liste des tickets détenus dans tickets. Cette liste est _synchronisée_ (comme pour tous les sous-documents).

#### Arrêtés mensuels
Le GC effectue des arrêtés mensuels consultables par le Comptable et l'administrateur technique. Chaque arrêté mensuel,
- récupère tous les tickets générés à M-3 (par exemple `202407`) et les efface de la liste `tickets`,
- les stocke dans un _fichier_ **CSV** `T_202407` du Comptable. Ces fichiers sont cryptés par la clé E de l'espace connue de l'administrateur et du Comptable.

Pour rechercher un ticket particulier, par exemple pour traiter un _litige_ ou vérifier s'il a bien été réceptionné, le Comptable,
- dispose de l'information en ligne pour tout ticket de M M-1 M-2,
- dans le cas contraire, ouvre l'arrêté mensuel correspondant au mois du ticket cherché qui est un fichier CSV basique.

#### Numérotation des tickets
L'ids d'un ticket est un string de la forme : `aammrrrrrrrrrr`
- `aa` : année de génération,
- `mm` : mois de génération,
- `r...r` : aléatoire.

Un code à 6 lettres majuscules en est extrait afin de le joindre comme référence de _paiement_.
- la première lettre  donne le mois de génération du ticket : A-L pour les mois de janvier à décembre si l'année est paire et M-X pour les mois de janvier à décembre si l'année est impaire.
- les autres lettres correspondent à `r...r`.

Le Comptable sait ainsi dans quel _arrêté mensuel_ il doit chercher un ticket au delà de M+2 de sa date de génération à partir d'un code à 6 lettres désigné par un compte pour audit éventuel de l'enregistrement.

> **Personne, pas même le Comptable,** ne peut savoir quel compte "A" a généré quel ticket. Cette information n'est accessible qu'au compte lui-même et est cryptée par sa clé K (la base connaît cette information mais elle est cryptée par la clé du site).

# Documents `chats`

Un chat est une suite d'items de texte communs à deux avatars I et E:
- vis à vis d'une session :
  - I est l'avatar _interne_,
  - E est un avatar _externe_ connu comme _contact_.
- un item est défini par :
  - le côté qui l'a écrit (I ou E),
  - sa date-heure d'écriture qui l'identifie pour son côté,
  - sa date-heure de suppression s'il a été supprimé.
  - son texte crypté par la clé de cryptage du chat connue seulement par I et E.

Un chat est dédoublé avec un exemplaire I et un exemplaire E:
- à son écriture, un item est ajouté des deux côtés.
- le texte d'un item écrit par I peut être effacé par I des deux côtés (mais pas modifié).
- I (resp. E) **peut effacer tous les items** I comme E de son côté: ceci n'impacte pas l'existence de ceux de l'autre côté.
- _de chaque côté_ la taille totale des textes de tous les items est limitée à 5000c. Les plus anciens items sont effacés afin de respecter cette limite.

Pour ajouter un item sur un chat, I doit connaître la clé de E : membre d'un même groupe, chat avec un autre avatar du compte, ou l'ayant obtenu depuis la phrase de contact de E.

## Clé d'un chat
La clé C du chat est générée à la création du chat et l'ajout du premier item:
- côté I, cryptée par la clé K de I,
- côté E, cryptée par la clé `pub` de E.

## Décompte des nombres de chats par compte
- un chat est compté pour 1 pour I quand la dernière opération qu'il a effectuée est un ajout: si cette dernière opération est un _classement en indésirable_, le chat est compte pour 0.
- ce principe de gestion évite de pénaliser ceux qui reçoivent des chats non sollicités et qui les _déclare indésirables_.

## Résiliation / disparition de E
Quand un avatar ou un compte s'auto-résilie ou quand le GC détecte la disparition d'un compte par dépassement de sa date limite de validité, il _résilie_ tous ses avatars, puis le compte lui-même.

A la résiliation d'un avatar,
- tous ses chats sont accédés et l'exemplaire de E l'est aussi:
- s'il était _indésirable_, il devient _zombi_, n'a plus de _data_.
- sinon, son statut `st` passe à 2. E conserve le dernier état de l'échange, mais,
  - il ne pourra plus le changer, la carte de visite de I reste dans le dernier état connu,
  - il ne pourra plus qu'effectuer un _classement en indésirable_, ce qui rendra l'exemplaire de son chat _zombi_.

## _data_ d'un chat
L'`id` d'un exemplaire d'un chat est le couple `id, ids`.

_data_ (de l'exemplaire I):
- `id`: id de I,
- `ids`: aléatoire.
- `v`: 1..N.
- `vcv` : version de la carte de visite de E.

- `st` : deux chiffres `I E`
  - I : 0:indésirable, 1:actif
  - E : 0:indésirable, 1:actif, 2:disparu
- `mutI` :
  - 1 - I a demandé à E de le muter en compte "O"
  - 2 - I a demandé à E de le muter en compte "A"
- `mutE` :
  - 1 - E a demandé à I de le muter en compte "O"
  - 2 - E a demandé à I de le muter en compte "A"
- `idE idsE` : identifiant de _l'autre_ chat.
- `cvE` : `{id, v, ph, tx}` carte de visite de E au moment de la création / dernière mise à jour du chat (textes cryptés par sa clé A).
- `cleCKP` : clé C du chat cryptée,
  - si elle a une longueur inférieure à 256 bytes par la clé K du compte de I.
  - sinon cryptée par la clé RSA publique de I.
- `cleEC` : clé A de l'avatar E cryptée par la clé du chat.
- `items` : liste des items `[{a, dh, l t}]`
  - `a` : 0:écrit par I, 1: écrit par E
  - `dh` : date-heure d'écriture.
  - `dhx` : date-heure de suppression.
  - `t` : texte crypté par la clé C du chat (vide s'il a été supprimé).

## Création d'un chat
**Sur création d'un compte par sponsoring**
- vérification qu'il existe un sponsoring créé par `idE` et qu'il accepte le chat.
- si le sponsorisé l'a souhaité.
- le chat est créé avec deux items: a) le mot de bienvenue du sponsor, b) la réponse du sponsorisé.

**Quand E est délégué de la partition de I**
- vérification que E est bien un délégué de la partition citée.
- I demande la clé RSA publique de E pour crypter la clé générée du chat.
- création avec un item.

**Quand E est membre d'un groupe G cité et I aussi**
- vérification que I accède aux membres de G et que E y est membre actif.
- I demande la clé RSA publique de E pour crypter la clé générée du chat.
- création avec un item.

**Quand I connaît la phrase de contact de E**
- vérification que E a bien cette phrase de contact et récupération de la cleA de E.
- I a calculé les hash des phrases de contact complète et réduite de E et obtenu en retour la clé A de E cryptée par le PBKFD de cette phrase de contact complète.
- I demande la clé RSA publique de E pour crypter la clé générée du chat.
- création avec un item.

Le nombre de chats dans la compta de I est incrémenté.

## Actions possibles (par I)
- _ajout d'un item_
  - l'item apparaît dans `items` de E (son `a` est inversé).
- _effacement du texte d'un item de I_
  - le texte de l'item est effacé des deux côtés.
  - il n'est pas possible pour I d'effacer le texte d'un item écrit par E.
- _déclarer indésirable_ : effacement total de l'historique des items (du côté I)
  - `items` est vidée du côté I.
  - `st` de I vaut `01` et `st` de E vaut `10` ou `00`.
  - le chat devient _indésirable_ du côté I.
- _faire un don_:
  Un compte "A" _donateur_ peut faire un don à un autre compte "A" _bénéficiaire_ en utilisant un chat.
  - le chat avec don ne peut intervenir que si le chat est défini entre les deux avatars **principaux** des comptes.
  - le montant du don est dans une liste préétablie.
  - le solde du donateur (dans sa `comptas`) doit être supérieur au montant du don.
  - sauf spécification contraire du donateur, le texte de l'item ajouté dans le chat à cette occasion mentionne le montant du don.
  - le donateur est immédiatement débité.
  - le bénéficiaire est immédiatement crédité dans `solde` de sa `comptas`.

> Un chat _indésirable_ pour un avatar reste un chat _écouté_, les items écrits par E arrivent, mais sur lequel I n'écrit pas. Il redevient _actif_ pour I dès que I écrit un item et ne redevient _indésirable_ que quand il fait une _déclaration d'indésirable_.

## Mutation des comptes O<->AR
### Compte "O"
Le compte doit donner son autorisation à un ou plusieurs comptes Comptable / délégués de sa partition. Pour chacun:
- le chat avec lui est marqué `mutI / mutE` 1 (autorisation de mutation en compte A).
- l'`ids` du chat est ajouté à `lmut` de son compte.

### Compte "A"
Le compte doit donner son autorisation à un ou plusieurs comptes Comptable / délégués _d'une_ partition. Pour chacun:
- le chat avec lui est marqué `mutI / mutE` 2 (autorisation de mutation en compte "O").
- l'`ids` du chat est ajouté à `lmut` de son compte.

### Pour les Comptable / délégués (_d'une_ partition)
Les actions de mutation sont accessibles depuis un _chat_ ayant un `mutE` à 1 ou 2. L'action de mutation, en plus de muter le compte,
- liste tous les chats de `lmut` du compte,
- efface les `mutI` de ces chats,
- supprime `lmut`.

L'opération d'auto-mutation d'un délégué en compte A est plus simple.

# Documents `sponsorings`

P est le parrain-sponsor, F est le filleul-sponsorisé.

_data_:
- `id` : id de l'avatar sponsor.
- `ids` : `hYR` hash du PBKFD de la phrase réduite de parrainage (précé en base du `ns`), 
- `v`: 1..N.
- `dlv` : date limite de validité

- `st` : statut. _0: en attente réponse, 1: refusé, 2: accepté, 3: détruit / annulé_
- `pspK` : texte de la phrase de sponsoring cryptée par la clé K du sponsor.
- `YCK` : PBKFD de la phrase de sponsoring cryptée par la clé K du sponsor.
- `hYC` : hash du PBKFD de la phrase de sponsoring,
- `dh`: date-heure du dernier changement d'état.
- `cleAYC` : clé A du sponsor crypté par le PBKFD de la phrase complète de sponsoring.
- `partitionId`: id de la partition si compte "O"
- `clePYC` : clé P de sa partition (si c'est un compte "O") cryptée par le PBKFD de la phrase complète de sponsoring (donne le numéro de partition).
- `nomYC` : nom du sponsorisé, crypté par le PBKFD de la phrase complète de sponsoring.
- `del` : `true` si le sponsorisé est délégué de sa partition.
- `cvA` : `{ id, v, ph, tx }` du sponsor, textes cryptés par sa cle A.
- `quotas` : `[qc, q1, q2]` quotas attribués par le sponsor.
  - pour un compte "A" `[0, 1, 1]`. Un tel compte n'a pas de `qc` et peut changer à loisir `[q1, q2]` qui sont des protections pour lui-même (et fixe le coût de l'abonnement).
- `don` : pour un compte autonome, montant du don.
- `dconf` : le sponsor a demandé à rester confidentiel. Si oui, aucun chat ne sera créé à l'acceptation du sponsoring.
- `dconf2` : le sponsorisé a demandé à rester confidentiel. Si oui, aucun chat ne sera créé à l'acceptation du sponsoring (ignoré en session UI).
- `ardYC` : ardoise de bienvenue du sponsor / réponse du sponsorisé cryptée par le PBKFD de la phrase de sponsoring.

**Remarques**
- la `dlv` d'un sponsoring peut être modifiée tant que le statut est _en attente_.
- Le sponsor peut annuler son `sponsoring` avant acceptation, en cas de remord son statut passe à 3.

**Si le sponsorisé refuse le sponsoring :** 
- Il écrit dans `ardYC` la raison de son refus et met le statut du `sponsorings` à 1.

**Si le sponsorisé ne fait rien à temps :** 
- `sponsorings` finit par être purgé par `dlv`.

**Si le sponsorisé accepte le sponsoring :** 
- Le sponsorisé crée son compte:
  - donne les hXR et hXC issus de sa phrase secrète,
  - génère ses clés K et celle de son avatar principal,
  - donne le texte de carte de visite.
- pour un compte "O", l'identifiant de la partition à la quelle le compte est associé est obtenu de `clePYC`.
- la `comptas` du sponsorisé est créée et créditée des quotas attribués par le sponsor pour un compte "O" et d'un `solde` minimum pour un compte "A".
- pour un compte "O" le document `partitions` est mis à jour (quotas attribués), le sponsorisé est mis dans la liste des comptes `tcpt` de `partitions`.
- un mot de remerciement est écrit par le sponsorisé au sponsor sur `ardYC` **ET** ceci est dédoublé dans un chat sponsorisé / sponsor créé à ce moment et comportant l'item de réponse. Si le sponsor ou le sponsorisé ont requis la confidentialité, le chat n'est pas créé.
- le statut du `sponsoring` est 2.

# Documents `notes`

La clé de cryptage d'une note est selon le cas :
- *note personnelle d'un avatar A* : la clé K de l'avatar.
- *note d'un groupe G* : la clé du groupe G.

Pour une note de groupe, le droit de mise à jour d'une note d'un groupe est contrôlé par `im` qui indique quel membre (son `im`) a l'exclusivité d'écriture (sinon tous).

_data_:
- `id` : id de l'avatar ou du groupe.
- `ids` : identifiant aléatoire universel.
- `v` : 1..N.

- `im` : exclusivité dans un groupe. L'écriture est restreinte au membre du groupe d'indice `im`. 
- `vf` : volume total des fichiers attachés.
- `ht` : liste des hashtags _personnels_ cryptée par la clé K du compte.
  - En session, pour une note de groupe, `ht` est le terme de `htm` relatif au compte de la session.
- `htg` : note de groupe : liste des hashtags cryptée par la clé du groupe.
- `htm` : note de groupe seulement, hashtags des membres. Map:
    - _clé_ : id du compte de l'auteur,
    - _valeur_ : liste des hashtags cryptée par la clé K du compte.
    - non transmis en session.
- `l` : liste des _auteurs_ (leurs `im`) pour une note de groupe.
- `d` : date-heure de dernière modification du texte.
- `texte` : texte (gzippé) crypté par la clé de la note.
- `mfa` : map des fichiers attachés.
- `pid pids` : référence de sa note _parent_.

**Une note peut être logiquement supprimée**. Afin de synchroniser cette forme particulière de mise à jour le document est conservé _zombi_ (sa _data_ est `null`). La note sera purgée un jour avec son avatar / groupe.

**Pour une note de groupe**, la propriété `htm` n'est pas transmise en session: l'item correspondant au compte est copié dans `ht`.

## Map des fichiers attachés
- _clé_ `idf`: identifiant aléatoire généré à la création. L'identifiant _externe_ est `id` du groupe / avatar, `idf`. En pratique `idf` est un identifiant absolu.
- _valeur_ : `{ idf, lg, ficN }`
  - `ficN` : `{ nom, info, dh, type, gz, lg, sha }` crypté par la clé de la note

**Identifiant de stockage :** `org/id/idf`
- `org` : code de l'organisation.
- `id` : id de l'avatar / groupe auquel la note appartient.
- `idf` : identifiant du fichier.

En imaginant un stockage sur file-system,
- l'application a un répertoire racine par espace portant le code de l'organisation,
- il y un répertoire par avatar / groupe ayant des notes ayant des fichiers attachés,
- pour chacun, un fichier par fichier attaché.

_Un nouveau fichier attaché_ est stocké sur support externe **avant** d'être enregistré dans son document `notes`. Ceci est noté dans un document `transferts`. 
Les fichiers créés par anticipation et non validés dans un document `notes` comme ceux qui n'y ont pas été supprimés après validation de la note, sont retrouvés par le GC.

La purge d'un avatar / groupe s'accompagne de la suppression de son _répertoire_. 

La suppression d'une note s'accompagne de la suppressions de N fichiers dans un seul _répertoire_.

## Note rattachée à une autre
Le rattachement d'une note à une autre permet de définir un _arbre_ des notes.
- une note d'un avatar A1 peut être rattachée:
  - soit à la racine A1, en fait elle n'est pas rattachée,
  - soit à une autre note de A1,
  - soit à un groupe G1: A1 peut ainsi commenter un groupe par des notes qu'il sera seul à voir.
  - soit à une note de G1: A1 peut ainsi commenter des notes d'un groupe par des notes qu'il sera seul à voir.
- une note d'un groupe G1 ne peut être rattachée qu'à une autre note du même groupe.

Les cycles (N1 rattachée à N2 rattachée à N3 rattachée à N1 par exemple) sont détectés et bloqués.

**A propos de `[pid, pids]`**:
- Pour un note de groupe:
  - `[null, null]`: rattachement _virtuel_ au groupe lui-même.
  - `[pid, pids]` : 
    - `pid`: ID du groupe lui-même (`id` de la note), 
    - `ids`: de la note du groupe à laquelle elle est rattachée (possiblement supprimée).
- Pour un note personnelle:
  - `[null, null]`: rattachement _virtuel_ à l'avatar de la note.
  - `[pid null]` : 
    - `pid`: ID d'UN GROUPE, 
    - rattachement _virtuel_ au groupe lui-même, possiblement disparu / radié.
  - `[pid, pids]`: 
    - SI `pid` est ID de l'avatar (de la note), 
      - `ids`: de la note de l'avatar à laquelle elle est rattachée (possiblement supprimée).
    - SI `pid`: est l'ID d'UN GROUPE, possiblement disparu / radié.
      - `ids`: de la note de ce groupe à laquelle elle est rattachée (possiblement supprimée).

# Documents `groupes`

Un groupe est caractérisé par :
- son entête : un document `groupes`.
- son (unique) sous-document `chatgrs` (dont `ids` est `1`).
- ses membres: des documents de sa sous-collection `membres`.

**Droits d'accès d'un membre.**

Um membre peut se _restreindre_ lui-même les accès suivants:
- [AM] : accès aux membres et au chat du groupe.
- [AN] : accès aux notes du groupe.

Il peut avoir les deux (cas général) ou n'en avoir aucun ce qui:
- limite son accès au groupe à la lecture de la carte de visite du groupe.
- pour un un animateur il peut être _hébergeur_.

Des droits d'accès sont conférés par un animateur (indépendamment de [AM] / [AN]):
- [DM] **d'accès à la liste des membres**.
- [DN] **d'accès en lecture aux notes du groupe**.
- [DE] **droits d'écriture sur les notes du groupe** (ce qui implique DN).

L'accès [AM] ([AN]):
- vrai: DES QUE le membre a le droit [DM], il accède aux membres. 
- faux: il n'accède pas aux membres **même s'il en le droit** par [DM].

L'historique synthétique est consigné par:
- [HM] **a eu un jour accès aux membres**
- [HN] **a eu un jour accès aux notes**
- [HE] **a eu un jour la possibilité d'écrire une note**

## Statut d'un membre dans le groupe: tables `st tid flags`
Ces trois tables sont synchrones: l'indice `im` d'un membre est le même pour les trois:
- `tid` : table des ids des membres.
- `st` : statut de ce membre.
- `flags`: accès et droits d'accès de ce membre.

> Ces tables s'étendent, les indices devenus inutiles ne sont pas réutilisés.

**Statut `st`:**
- 0 : **radié**: c'est un ex-membre désormais _inconnu_, peut-être disparu, son id est perdu (`tid[im]` vaut null).
- 1 : **contact** proposé pour une éventuelle invitation future par un membre ayant un droit d'accès aux membres.
  - l'avatar n'est pas au courant et ne peut rien faire dans le groupe.
  - les membres du groupe peuvent voir sa carte de visite.
  - un animateur peut le faire passer en état _invité_ ou _le radier_ (avec ou sans inscription en liste noire _groupe_).
- 2 / 3 : **pré-invité / invité**: 
  - 2 : **pré-invité : en attente de votes** quand un vote unanime est requis. Un ou plusieurs animateurs ont voté pour inviter le contact, mais pas tous.
    - l'avatar n'est pas au courant et ne peut rien faire dans le groupe.
    - les membres du groupe peuvent voir sa carte de visite.
    - les animateurs peuvent:
      - voter _pour_, effacer leur vote, changer les conditions d'invitation. Quand tous les animateurs ont voté pour, l'avatar devient _invité_.
      - _le radier_ (avec ou sans inscription en liste noire _groupe_) ou annuler l'invitation et le conserver comme simple contact.
  - 3 : **invité: en attente de réponse** quand le dernier animateur a voté (ou le premier pour les groupes à invitation simple), l'invitation a été transmise à l'avatar invité.
    - l'avatar proposé est au courant, il a une _invitation_ dans son compte (`invits`).
    - les membres du groupe peuvent voir sa carte de visite et les droits d'accès qui seront appliqués si l'avatar accepte l'invitation.
    - un animateur peut:
      - changer ses droits d'accès futurs.
      - _le radier_ (avec inscription en liste noire _groupe_) ou annuler l'invitation et le conserver comme simple contact.
    - l'avatar peut,
      - accepter l'invitation: il passera en état _actif_ ou _animateur_.
      - refuser l'invitation et _s'auto-radier_ (avec ou sans inscription en liste noire _compte_) ou redevenir simple contact.
- 4 / 5 : **actif** 
  - 4 : **non animateur**
    - l'avatar a le groupe enregistré dans son compte (`mpg`).
    - il peut:
      - changer son accès aux membres et aux notes (mais pas ses droits).
      - _s'auto-radier_ (avec on sans inscription en liste noire _compte_) ou redevenir simple contact.
    - un animateur peut:
      - changer ses droits d'accès (mais pas ses accès effectifs qui sont du ressort du membre).
      - le radier, avec ou sans inscription en en liste noire _groupe_ ou simplement le ramener en statut _contact_.
  - 5 : **animateur**. _actif_ avec privilège d'animation.
    - un autre animateur ne peut ni changer ses droits, ni le radier, ni lui retirer son statut d'animateur (mais lui-même peut le faire).

Le nombre de notes pris en compte dans la comptabilité du compte:
- est incrémenté de 1 quand il accepte une invitation,
- est décrémenté de 1 quand il est radié ou redevient simple contact.

Dès qu'un membre a un statut:
- un indice `im` lui est attribué en séquence du dernier attribué (taille de `tid`). 
- ses accès et droits sont consignés dans la table `flags` à l'indice `im`.
- il a un document `membres` associé: `ids`, l'identification relative du membre dans le groupe est son indice `im`.

## Radiations et inscriptions en liste noires `lng lnc`
La liste noire `lng` est la liste des ID des membres que l'animateur ne veut plus voir réapparaître dans le groupe après leur radiation.

La liste noire `lnc` est la liste des ID des membres qui se sont auto-radiés en indiquant ne jamais vouloir être connu du groupe à l'avenir.

Un animateur peut radier un membre, sauf les autres animateurs.

Un membre actif peut _s'auto-radier_:
- il ne verra plus le groupe dans sa liste des groupes.
- sans inscription en liste noire, il pourra ultérieurement être réinscrit comme contact ou réinvité comme s'il n'avait jamais participé au groupe.
- avec inscription en liste noire il ne pourra plus jamais ultérieurement être réinscrit comme contact ou réinvité.

A la radiation d'un membre d'indice `im`:
- son document `membres` est logiquement détruit (passe en _zombi_).
- il peut être inscrit dans les listes noires `lng lnc`.
- ses entrées dans `tid st flags` sont à `null 0 0`.

Un membre actif peut décider de redevenir _simple contact_:
- il ne verra plus le groupe dans sa liste des groupes.
- une trace historique simplifiée de son existence subsiste: 
  - dans ses flags (HM HN HE), 
  - dans les dates importantes dans son document membre.

> Quand le GC découvre la _disparition_ du compte d'un avatar membre, il s'opère l'équivalent d'une radiation sans mise en liste noire (mais l'avatar ne reviendra jamais).

## Création d'un membre
Le membre _fondateur_ du groupe a un _indice_ `im` 1 et est créé au moment de la création du groupe:
- dans la table `flags` à l'indice `im`: `DM DN DE AM AN HM HN HE`
  - il a _droit_ d'accès aux membres et aux notes en écriture,
  - ses accès aux membres et notes sont ouverts,
  - il a pour statut `st[1]` 5 : _animateur_.
- son id figure en `tid[1]`.

Les autres membres sont créés, lorsqu'ils sont soit proposés comme contact, soit invités.
- un indice `im` est pris en séquence, `tid[im]` contient leur ID.
- leur document `membres` est créé. 
- _proposition de contact_: leurs flags sont à 0, son statut est à 1.
- _invitation_: 
  - des flags donnent les _droits_ futurs DM DN DE et _animateur_ selon le choix de l'animateur.
  - une **invitation** est insérée dans leur avatar.

> Réapparition d'un membre après _radiation sans liste noire_ par un animateur.
Un animateur peut radier un avatar sans le mettre en liste noire. L'avatar peut être réinscrit comme contact / réinvité plus tard et aura un nouvel indice et un nouveau document `membres`, son historique est vierge. 

## Modes d'invitation
- _simple_ : dans ce mode (par défaut) un _contact_ du groupe peut-être invité par **UN** animateur (un seul suffit).
- _unanime_ : dans ce mode il faut que **TOUS** les animateurs aient validé l'invitation (le dernier ayant validé provoquant l'invitation).
- pour passer en mode _unanime_ il suffit qu'un seul animateur le demande.
- pour revenir au mode _simple_ depuis le mode _unanime_, il faut que **TOUS** les animateurs aient validé ce retour.

Une invitation est enregistrée dans la liste `invits` du compte de l'avatar invité:
- `invits` du document `invits`: `[{idg, ida, cleGA, cvG, invpar, txtG}]`
  - `idg`: id du groupe,
  - `ida`: id de l'avatar
  - `cleGA`: clé du groupe crypté par la clé A de l'avatar.
  - `cvG` : carte de visite du groupe (photo et texte sont cryptés par la clé G du groupe).
  - `flags` : d'invitation. Animateur DM DN DE.
  - `invpar` : `[{ cleAG, cvA }]`
    - `cleAG`: clé A de l'avatar invitant crypté par la clé G du groupe.
    - `cvA` : carte de visite de l'invitant (photo et texte sont cryptés par la clé G du groupe). 
  - `msgG` : message de bienvenue / invitation émis par l'invitant.

Ces données permettent à l'invité de voir en session les cartes de visite du groupe et du ou des invitants ainsi que le texte d'invitation (qui figure également dans le chat du groupe). Le message de remerciement en cas d'acceptation ou de refus sera également inscrit dans le chat du groupe.

### Invitations en cours: `invits`
Cette map de `groupes` a une entrée par invitation _ouverte_ et pas encore ni acceptée ni refusée ni même totalement votée: `{ fl, li[] }`
- `fl` : flags d'invitation. Droits futurs DM DN DE et pouvoir d'animation.
- `li` :
  - liste des `im` des animateurs ayant voté l'invitation pour un mode unanime.
  - pour un mode d'invitation simple, il n'y a qu'un terme.

Quand l'invitation a été acceptée ou refusée, l'entrée correspondante dans `invits` est détruite.

Quand l'invitation est encore _en vote_ (statut 2), les listes `li` sont remises à jour quand un des animateurs cités n'est plus _actif animateur_.

## Un membre peut avoir plusieurs périodes d'activité
- il est inscrit une fois comme _contact_ puis est _invité_.
- il accepte l'invitation et devient actif.
- il décide de redevenir _simple contact_ (sans se radier): sa période d'activité se termine.
- il est à nouveau _invité_ et accepte son invitation: sa deuxième période d'activité commence.

Tant que le membre,
- ne s'est pas auto-radié,
- n'a pas été radié par un animateur,
- il conserve son indice `im`, son document `membres` et une trace des périodes d'activité:
  - ses flags `HM HN HE` indiquent sommairement s'il a eu _un jour_ accès aux membres, accès aux notes en lecture ou en écriture.
  - dans son document `membres`, les couples de dates de début de la première période et de fin de la dernière période d'activité, d'accès aux membres et aux notes en lecture ou écriture.

## Hébergement par un membre _actif_
L'hébergement d'un groupe est noté par :
- `imh`: indice membre de l'avatar hébergeur. 
- `idh` : id du **compte** de l'avatar hébergeur. **Cette donnée n'est pas transmise aux sessions**.
- `dfh`: date de fin d'hébergement qui vaut 0 tant que le groupe est hébergé. Les notes ne peuvent plus être mises à jour _en croissance_ quand `dfh` existe.

### Prise d'hébergement
- en l'absence d'hébergeur, c'est possible pour,
  - tout animateur,
  - en l'absence d'animateur: tout actif ayant le droit d'écriture des notes, puis tout actif ayant accès aux notes, puis tout actif.
- s'il y a déjà un hébergeur, seul un animateur peut se substituer à l'hébergeur actuel.
- dans tous les cas c'est à condition que le nombre de notes `nn` et le volume de fichiers actuels `vf` ne le mette pas en dépassement de son abonnement.

### Fin d'hébergement par l'hébergeur
- `dfh` est mise la date du jour + 90 jours.
- le nombre de notes `ng` et le volume `v` de `comptas` sont décrémentés de ceux du groupe.

Au dépassement de `dfh`, le GC détruit le groupe.

## Data
_data_:
- `id` : id du groupe.
- `v` :  1..N
- `dfh` : date de fin d'hébergement.

- `alias` : ID alternative générée à la création du groupe. Identifie le groupe sur Storage pour éviter d'y faire figurer son ID réelle.
- `nn qn vf qv`: nombres de notes actuel et maximum autorisé par l'hébergeur, volume total actuel des fichiers des notes et maximum autorisé par l'hébergeur.
- `idh` : id du compte hébergeur (pas transmise aux sessions).
- `imh` : indice `im` du membre dont le compte est hébergeur.
- `msu` : mode _simple_ ou _unanime_.
  - `null` : mode simple.
  - `[im]` : mode unanime : liste des indices des animateurs ayant voté pour le retour au mode simple. La liste peut être vide mais existe.
- `invits` : map `{ fl, li[] }` des invitations en attente de vote ou de réponse. Clé: `im` du membre invité.
- `tid` : table des IDs des membres.
- `st` : table des statuts.
- `flags` : tables des flags.
- `lng` : liste noire _groupe_ des ids (courts) des membres.
- `lnc` : liste noire _compte_ des ids (courts) des membres.
- `cvG` : carte de visite du groupe, textes cryptés par la clé du groupe `{v, photo, info}`.

## Décompte des participations à des groupes d'un compte
- quand un avatar a accepté une invitation, il devient _actif_ et a une nouvelle entrée dans la liste des participations aux groupes (`mpg`) dans l'avatar principal de son compte.
- quand l'avatar est radié cette entrée est supprimée.
- le _nombre de participations aux groupes_ dans `comptas.qv.ng` du compte est le nombre total de ces entrées dans `mpg`.

# Documents `membres`
Un document `membres` est créé à la déclaration d'un avatar comme _contact_.

Le document `membres` est détruit,
- par une opération de radiation.
- par la destruction de son groupe lors de la résiliation du dernier membre actif.

_data_:
- `id` : id du groupe.
- `ids`: indice `im` de membre relatif à son groupe.
- `v` : 
- `vcv` : version de la carte de visite du membre.

- `dpc` : date de premier contact.
- `ddi` : date de la dernière invitation (envoyée au membre, c'est à dire _votée_).
- **dates de début de la première et fin de la dernière période...**
  - `dac fac` : d'activité.
  - `dln fln` : d'accès en lecture aux notes.
  - `den fen` : d'accès en écriture aux notes.
  - `dam fam` : d'accès aux membres.
- `cleAG` : clé A de l'avatar membre cryptée par la clé G du groupe.
- `cvA` : carte de visite du membre `{id, v, ph, tx}`, textes cryptés par la clé A de l'avatar membre.
- `msgG`: message d'invitation crypté par la clé G pour une invitation en attente de vote ou de réponse. 

> Un message d'invitation est aussi inscrit dans le chat du groupe ou figure aussi la réponse de l'invité. `msgG` est effacé après acceptation ou refus, mais pas les items correspondants dans le chat.

## Opérations
### Par un animateur:
- Inscription en contact - 0 -> 1

- Radiation d'un contact - 1 -> 0
- Invitation simple - 1 -> 3
- Annulation d'invitation - 2 / 3
  - et retour en contact -> 1
  - et radiation sans inscription en liste noire G -> 0
  - et radiation avec inscription en liste noire G -> 0
- Vote d'invitation - 2 
  - vote pour -> 2 ou 3
  - retrait d'un vote pour
- Modification des conditions d'invitation - 2 / 3. Pour le mode _unanime_ revient à 2 (votes annulés)
- Modification des droits d'accès - 4 / 5
- Radiation d'un membre actif - 4 / 5
  - et retour en contact -> 1
  - et radiation sans inscription en liste noire G -> 0
  - et radiation avec inscription en liste noire G -> 0

### Par le membre lui-même:
- Acceptation d'invitation - 3 -> 4 / 5
- Refus d'une invitation - 3 
  - et retour en contact -> 1
  - et radiation sans inscription en liste noire C -> 0
  - et radiation avec inscription en liste noire C -> 0
- Modification des accès membre / note et statut d'animateur - 4 / 5 -> 4 (si retrait animateur)
- Auto-radiation - 4 / 5
  - et retour en contact -> 1
  - et radiation sans inscription en liste noire C -> 0
  - et radiation avec inscription en liste noire C _> 0

### Inscription en contact
- s'il est en liste noire, refus.
- attribution de l'indice `im`.
- un row `membres` est créé.

### Invitation par un animateur
- choix des _droits_ et inscription dans `invits` du compte de l'avatar.
- vote d'invitation (en mode _unanime_):
  - si tous les animateurs ont voté, inscription dans `invits` du compte de l'avatar.
  - si le votant change les _droits_, les autres votes sont annulés.
- `ddi` est remplie dans `membres`.

### Annulation d'invitation par un animateur
- effacement de l'entrée de l'id du groupe dans `invits` du compte de l'avatar.

### Radiation par un animateur (avec ou sans liste noire)
- le statut passe de 1-2 (sinon erreur) à 0.
- s'il était invité, effacement de l'entrée de l'id du groupe dans `invits` de l'avatar.
- inscription éventuelle en liste noire `lng`.
- le document `membres` devient _zombi_.

### Refus d'invitation par le compte
- mise à 0 du statut, des flags et de l'entrée dans `tid`.
- document `membres` mis en _zombi_
- Option liste noire: inscription dans `lnc`.
- son item dans `invits` du compte de son avatar est effacé.

### Acceptation d'invitation par le compte
- dans l'avatar principal du compte un item est ajouté dans `mpg`,
- dans `comptas` le compteur `qv.ng` est incrémenté.
- `dac dln ... fam` de `membres` sont mises à jour.
- son item dans `invits` du compte de son avatar est effacé.
- flags `AN AM`: accès aux notes, accès aux autres membres.
- statut à 3 ou 4.

### Modification des droits par un animateur
- flags `DM DN DE`

### Radiation d'un actif par un animateur
- le statut est actif et deviendra 0 (radié) ou 1 (retour en contact).
- le membre est mis (ou non) en liste noire `lng`.
- cas de radiation: son document `membres` est mis en _zombi_.

### Modification des accès membres / notes par le compte
- flags `AN AM`: accès aux notes, accès aux autres membres.

## Radiation demandée par le compte**
- document `membres` mis en _zombi_.
- mis à 0 du statut, de l'entrée dans tid. Dans flags il ne reste que les HM HN HE.
- si le membre était le dernier _actif_, le groupe disparaît.
- la participation au groupe disparaît de `mpg` du compte.
- option liste noire: mise en liste noire `lnc`.

# Documents `Chatgrs`

A chaque groupe est associé **UN** document `chatgrs` qui représente le chat des membres d'un groupe. Il est créé avec le groupe et disparaît avec lui.

_data_
- `id` : id du groupe
- `ids` : `1`
- `v` : sa version.

- `items` : liste ordonnée des items de chat `{im, dh, dhx, t}`
  - `im` : im du membre auteur,
  - `dh` : date-heure d'écriture.
  - `dhx` : date-heure de suppression.
  - `t` : texte gzippé crypté par la clé G du groupe (vide s'il a été supprimé).

## Opérations
### Ajout d'un item
- autorisé pour tout membre actif ayant droit d'accès aux membres.
- le texte est limité à 300 signes.

### Effacement d'un item
- autorisé pour l'auteur de l'item ou un animateur du groupe.

### Sur invitation par un animateur
- le texte d'invitation est enregistré comme item, les autres membres du groupe peuvent ainsi le voir.

### Sur acceptation ou refus d'invitation
- le texte explicatif est enregistré comme item.

Un item ne peut pas être corrigé après écriture, juste effacé.

Le chat d'un groupe garde les items dans l'ordre ante-chronologique jusqu'à concurrence d'une taille totale de 5000 signes.

# Résiliation d'un compte, avatar, groupe

Elle est effectuée en deux phases:
- **une transaction courte immédiate:**
  - récupère les groupes où l'avatar est invité (dans `invits` de son compte) et dont il est membre actif (dans `mpg` de son compte).
  - pour chacun de ces groupes, supprime ce qui est relié à cet avatar.
  - supprime l'avatar dans la liste des avatars `mav` de son compte.
  - met l'avatar en état _zombi_. Ceci marque le document `versions` de l'avatar à _supprimé_ (`suppr` porte la date du jour).
  - purge ses documents `sponsorings`.
  - dès lors l'avatar est _logiquement_ supprimé.
  - inscription des deux tâches différées `AVC` et `AGN`.
- **deux tâches différées sont lancées:**
  - `AVC`: met à jour les chats _externes_ / purge les chats _internes_ de l'avatar.
  - `AGN`: purge les notes de l'avatar.

La résiliation d'un avatar peut provoquer la suppression d'un groupe quand l'avatar était le dernier membre actif.

## Suppression d'un groupe
Elle est effectuée en deux phases:
- **dans l'opération / transaction principale:**
  - suppression des invitations en cours.
  - suppression du groupe des comptes l'ayant en tant que participant.
  - mise à jour de la comptas du compte hébergeur.
  - le groupe est mis à l'état _zombi_. Ceci marque le document `versions` du groupe à _supprimé_ (`suppr` porte la date du jour).
  - inscription des deux tâches différées `GRM` et `AGN`.
- **deux tâches différées sont lancées:**
  - `GRM`: purge les _membres_ du groupe.
  - `AGN`: purge les notes du groupe.

## Résiliation d'un compte
En une transaction la résiliation immédiate des avatars du compte est effectuée, ce qui lance une chaîne longue de transactions différées.

# Contrôle global des ressources d'un espace
Les quotas globaux sont attribués par l'administrateur technique.

Le Comptable attribue une _réserve_ globale pour tous les comptes A: la somme de leurs quotas ne pourra pas dépasser ce quota.
- le restant entre les quotas globaux et ceux réservés pour les comptes A, sont utilisables pour les partitions (compte O).

Le Comptable crée des partitions et leur alloue à chacune des quotas. La somme de ces quotas ne peut pas être supérieure aux quotas globaux diminués de ceux réservés pour les comptes A.

Le Comptable et les délégués d'une partition allouent des quotas aux comptes de la partition, leur somme ne peut pas dépasser les quotas de la partition.

Lorsqu'à un niveau donné, _global, réserve des A, partition_, les quotas sont révisés à la baisse, il n'y a pas d'impact immédiat. Toutefois les futures allocations / révisions de quotas du niveau inférieur ne peuvent intervenir qu'à la baisse, du moins sur le / les quotas _documents, volume de fichiers, calcul_ excédentaire.

### Dans `espaces`
L'attribut `quotas { qn, qv, qc }` donne les quotas globaux maximum attribués par l'administrateur technique.

La somme des quotas attribués aux partitions et pour les comptes A ne doit pas dépasser ces valeurs.

### Dans `syntheses`
- `qA` : `{ qc, qn, qv }` - quotas **maximum** disponibles pour les comptes A.
- `qtA` : `{ qc, qn, qv }` - quotas **effectivement attribués** aux comptes A. En conséquence `qA.qn - qtA.qn` est le quotas qn encore attribuable aux compte A.

- `tsp[p]`
  - `q` : `{ qc, qn, qv }` - quotas **maximum** de la partition p
  - `qt` : `{ qc qn qv c2m n v }` - quotas `qc qn qv` **effectivement attribués** aux comptes de la partition et consommations actuelles `c2m n v`(approximatives).
  - En conséquence q.qn - qt.qn par exemple représente les quotas encore attribuables aux comptes de la partition p.

`tsp['0']` est la totalisation des `tsp[p]` calculé à la compilation.
- `q` : `{qc, qn, qv}` sont les quotas totaux maximum pour l'ensemble des partitions.

Les quotas globaux réservés par le Comptable est la somme dans `syntheses` de `qA + tsp['0'].q` et elle doit toujours être inférieure à `espaces.quotas`.

### Opérations impactant ces compteurs
Attribution de quotas par l'administrateur technique (remplace l'attribution de profil).
- affecte `espaces.quotas`

Attribution / ajustement des quotas des comptes A par le Comptable
- affecte `syntheses.qA`
- bloqué si c'est en augmentation ET fait dépasser le quotas général de espaces.

Attribution / ajustement des quotas d'une partition p par le Comptable
- affecte `partition[p].q` ce qui se répercute sur `syntheses.tsp[p].q`
- bloqué si c'est en augmentation ET fait dépasser le quotas général de espaces.

Ajustement de ses quotas par un compte A dont un `qc` (qui peut le ralentir).
- affecte comptes[A].qv
- augmente / diminue `syntheses.qtA`
- bloqué si c'est en augmentation ET fait dépasser le quotas général de espaces.

Mutation d'un compte `c` A en compte O de la partition `p`
- diminue `syntheses.qtA`
- augmente `partition[p].mcpt[c].q` (si c'est possible) ce qui se répercute sur `syntheses.tsp[p].qt`
- blocage si les quotas de la partition ne supportent pas les quotas du compte muté.

Mutation d'un compte `c` O de la partition `p` en compte A
- augmente `syntheses.qtA`.
- diminue `partition[p].mcpt[c].q` ce qui se répercute sur `syntheses.tsp[p].qt`.
- bloqué si l'augmentation de `syntheses.qtA` fait dépasser `syntheses.qA`.

Nouveau sponsoring
- vérifie que les quotas alloués seraient acceptables si la création du compte avait lieu à cet instant.

Création du compte par sponsoring
- bloque si les quotas enregistrés dans le sponsoring ne peuvent pas être honorés, les conditions ayant changé entre l'enregistrement du sponsoring et l'instant de la création.

# Gestion des disparitions des comptes: `dlv` 

Chaque compte a une **date limite de validité**:
- toujours une _date de dernier jour du mois_ (sauf exception par convention décrite plus avant),
- propriété indexée de son document `comptes`.

Le GC utilise le dépassement de `dlv` pour libérer les ressources correspondantes (notes, chats, ...) d'un compte qui n'est plus utilisé:
- **pour un compte "A"** la `dlv` représente la limite d'épuisement de son crédit mais bornée à `nbmi` mois du jour de son calcul.
- **pour un compte "O"**, la `dlv` représente la plus proche de ces deux limites,
  - un nombre de jours sans connexion (donnée par `nbmi` du document `espaces` de l'organisation),
  - la date `dlvat` jusqu'à laquelle l'organisation a payé ses coûts d'hébergement à l'administrateur technique (par défaut la fin du siècle). C'est la date `dlvat` qui figure dans le document `espaces` de l'organisation. Dans ce cas, par convention, c'est la **date du premier jour du mois suivant** pour pouvoir être reconnue.

> Remarque. En toute rigueur un compte "A" qui aurait un gros crédit pourrait ne pas être obligé de se connecter pour prolonger la vie de son compte _oublié / tombé en désuétude / décédé_. Mais il n'est pas souhaitable de conserver des comptes _morts_ en hébergement, même payé: ils encombrent pour rien l'espace.

## Calcul de la `dlv` d'un compte
La `dlv` d'un compte est recalculée à plusieurs occasions.

### Acceptation du sponsoring du compte
Première valeur calculée selon le type du compte.

### Connexion
La connexion permet de refaire les calculs en particulier en prenant en compte de nouveaux tarifs.

C'est l'occasion majeure de prolongation de la vie d'un compte.

### Don pour un compte "A": passe par un chat
Les `dlv` du _donneur_ et du récipiendaire sont recalculées sur l'instant.

### Enregistrement d'un crédit par le Comptable
Pour le destinataire du crédit sa dlv est recalculée sur l'instant.

### Modification de l'abonnement d'un compte A
La `dlv` est recalculée à l'occasion de la nouvelle évaluation qui en résulte.

### Mutation d'un compte "O" en "A" et d'un compte "A" en "O"
La `dlv` est recalculée en fonction des nouvelles conditions.

## Changement des paramètres dans l'espace d'une organisation
Il y a deux données: 
- `dlvat`: date limite de vie des comptes "O", fixée par l'administrateur technique en fonction des contributions effectives reçues de l'organisation pour héberger ses comptes "O".
- `nbmi`: nombre de mois d'inactivité acceptable fixé par le Comptable (3, 6, 9, 12, 18 ou 24). Ce changement n'a pas d'effet rétroactif.

> **Remarque**: `nbmi` est fixé par configuration par le Comptable _pour chaque espace_. C'est une contrainte de délai maximum entre deux connexions à un compte, faute de quoi le compte est automatiquement supprimé. La constante `IDBOBS` fixe elle un délai maximum (2 ans par exemple), _pour un appareil et un compte_ pour bénéficier de la synchronisation incrémentale. Un compte peut se connecter toutes les semaines et avoir _un_ poste sur lequel il n'a pas ouvert une session synchronisée depuis 3 ans: bien que tout à fait vivant, si le compte se reconnecte en mode _synchronisé_ sur **ce** poste, il repartira depuis une base locale vierge, sans bénéficier d'un redémarrage incrémental.

### Changement de `dlvat`
Si le financement de l'hébergement par accord entre l'administrateur technique et le Comptable d'un espace tarde à survenir, beaucoup de comptes "O" ont leur existence menacée par l'approche de cette date couperet. Un accord tardif doit en conséquence avoir des effets immédiats une fois la décision actée.

Par convention une `dlvat` est fixée au **1 d'un mois** et ne peut pas être changée pour une date inférieure à M + 3 (nbmi ?) du jour de modification.

L'administrateur technique qui remplace une `dlvat` le fait en plusieurs transactions pour toutes les `dlv` des `comptes` égales à l'ancienne `dlvat`. La transaction finale fixe aussi la nouvelle `dlvat`. La valeur de remplacement est,
- la nouvelle `dlvat` (au 1 d'un mois) si elle est inférieure à `auj + nbmi mois`: c'est encore la `dlvat` qui borne la vie des comptes O (à une autre borne).
- sinon la fixe à `auj + nbmi mois` (au dernier jour d'un mois), comme si les comptes s'étaient connectés aujourd'hui.

_Remarque_: idéalement une transaction unique aurait été préférable mais elle pourrait être longue et entraînerait des blocages.

# Décomptes des coûts et crédits

> **Remarque**: en l'absence d'activité de sessions la _consommation_ d'un compte est nulle, alors que le _coût d'abonnement_ augmente à chaque seconde même sans activité.

On décompte **dans le service OP** le nombre de lectures et d'écritures effectués dans chaque opération:
- intégration dans le document `comptas` du compte, le cas échéant avec propagation au compte (voire partition) si le changement est significatif.
- le volume de _download_ est décompté quand une session demande une URL de _GET_ d'un fichier (en considérant que puisqu'elle a demandé l'URL, elle s'en est servi).
- le volume _d'upload_ est décompté sur l'opération qui valide l'upload. 
- retour à la session pour information où sont cumulés les 4 compteurs depuis le début de la session.

Le tarif de base par défaut repris pour les estimations est celui de Firebase [https://firebase.google.com/pricing#blaze-calculator]: ces tarifs sont de facto fixés dans `config.mjs`.

Le volume _technique_ moyen d'un groupe / note / chat est estimé à 8K. Ce chiffre est probablement faible, le volume _utile_ en Firestore étant faible par rapport au volume réel occupé avec les index ... D'un autre côté, le serveur considère les volumes utilisés en base alors que n / v vont être décomptés sur des quotas (des maximum rarement atteints).

## Classe `Tarif`
Un tarif correspond à,
- `am`: son premier mois d'application. Un tarif s'applique toujours au premier de son mois.
- `cu` : un tableau de 6 coûts unitaires `[u1, u2, ul, ue, um, ud]`
  - `u1`: 30 jours de quota qn (250 notes / chats)
  - `u2`: 30 jours de quota qv (100Mo)
  - `ul`: 1 million de lectures
  - `ue`: 1 million d'écritures
  - `um`: 1 GB de transfert montant.
  - `ud`: 1 GB de transfert descendant.

En configuration un tableau ordonné par `aaaamm` donne les tarifs applicables, ceux de plus d'un an n'étant pas utiles. 

L'initialisation de la classe `Tarif.init(...)` est faite depuis la configuration du service OP. Le tarif courant est communiqué à la connexion de la session afin d'éviter une divergence entre application Web et service OP.

On ne modifie pas les tarifs rétroactivement, en particulier celui du mois en cours (les _futurs_ c'est possible).

La méthode `const t = Tarif.cu(a, m)` retourne le tarif en vigueur pour le mois indiqué.

## Objet quotas et volumes `qv` : `{ qc, qn, qv, nn, nc, ng, v }`
- `qc`: quota de consommation
- `qn`: quota du nombre total de notes / chats / groupes.
- `qv`: quota du volume des fichiers.
- `nn`: nombre de notes existantes.
- `nc`: nombre de chats existants.
- `ng` : nombre de participations aux groupes existantes.
- `v`: volume effectif total des fichiers.

Cette objet est la propriété `qv` de `comptas`. 

## Objet consommation `conso` : `{ nl, ne, vm, vd }`
- `nl`: nombre absolu de lectures depuis la création du compte.
- `ne`: nombre d'écritures.
- `vm`: volume _montant_ vers le Storage (upload).
- `vd`: volume _descendant_ du Storage (download).

Cet objet rapporte une évolution de consommation.

## Unités
- T : temps.
- D : nombre de document (note, chat, participations à un groupe).
- B : byte.
- L : lecture d'un document.
- E : écriture d'un document.
- € : unité monétaire.

## Classe `Compteurs`
Cette classe donne les éléments de facturation et des éléments de statistique d'utilisation sur les les 12 derniers mois (mois en cours y compris).

**Propriétés:**
- `dh0` : date-heure de création du compte.
- `dh` : date-heure courante (dernier calcul).
- `qv` : quotas et volumes pris en compte au dernier calcul `{ qc, qn, qv, nn, nc, ng, v }`.
- Quand on _prolonge_ l'état actuel pendant un certain temps AVANT d'appliquer de nouvelles valeurs, il faut pouvoir disposer de celles-ci.
- `vd` : [0..3] - vecteurs détaillés pour M M-1 M-2 M-3.
- `mm` : [0..18] - coût abonnement + consommation pour le mois M et les 17 mois antérieurs (si 0 pour un mois, le compte n'était pas créé).
- `aboma` : somme des coûts d'abonnement des mois antérieurs au mois courant depuis la création du compte.
- `consoma` : somme des coûts de consommation des mois antérieurs au mois courant depuis la création du compte.

Le vecteur `vd[0]` et le montant `mm[0]` vont évoluer tant que le mois courant n'est pas terminé. Pour les mois antérieurs `vd[i]` et `mm[i]` sont immuables.

### Dynamique
Un objet de class `Compteurs` est construit,
- soit depuis `serial`, la sérialisation de son dernier état,
- soit depuis `null` pour un nouveau compte.
- la construction recalcule tout l'objet: il était sérialisé à un instant `dh`, il est recalculé pour être à jour à l'instant t.
- **puis** il peut être mis à jour, facultativement, juste avant le retour du `constructor`, par:
  - `qv` : quand il faut mettre à jour les quotas ou les volumes,
  - `conso` : quand il faut enregistrer une consommation.

`const compteurs = new Compteurs(serial, qv, conso, dh)`
- `dh` est facultatif et sert pour effectuer des batteries de tests ne dépendants pas de l'heure courante.

### Vecteur détaillé d'un mois
Pour chaque mois M à M-3, il y a un **vecteur** de 14 (X1 + X2 + X2 + 3) compteurs:
- moyennes et cumuls servent au calcul au montant du mois:
  - QC : moyenne de qc dans le mois (€)
  - QN : moyenne de qn dans le mois (D)
  - QV : moyenne de qv dans le mois (B)
  - NL : nb lectures cumulés sur le mois (L),
  - NE : nb écritures cumulés sur le mois (E),
  - VM : total des transferts montants (B),
  - VD : total des transferts descendants (B).
- compteurs de _consommation moyenne sur le mois_ qui n'ont qu'une utilité documentaire.
  - NN : nombre moyen de notes existantes.
  - NC : nombre moyen de chats existants.
  - NG : nombre moyen de participations aux groupes existantes.
  - V : volume moyen effectif total des fichiers stockés.
- 3 compteurs spéciaux
  - MS : nombre de ms dans le mois - si 0, le compte n'était pas créé
  - CA : coût de l'abonnement pour le mois
  - CC : coût de la consommation pour le mois

Le getter `get serial ()` retourne la sérialisation de l'objet afin de l'écrire dans la propriété `compteurs` de `comptas`.

**En session,** `compteurs` est recalculé par `compile()` à la connexion et en synchro,

**En serveur,** des opérations peuvent faire évoluer `qv` de `comptas`. L'objet `compteurs` est construit (avec un `qv` -et `conso` s'il enregistre une consommation) puis sa sérialisation est enregistrée dans `comptas`:
- création / suppression d'une note ou d'un chat: incrément / décrément de nn / nc.
- prise / abandon d'hébergement d'un groupe: delta sur nn / nc / v.
- création / suppression de fichiers: delta sur v.
- enregistrement d'un changement de quotas qn / qv.
- upload / download d'un fichier: delta sur vm / vd.
- enregistrement d'une consommation de calcul: delta sur nl / ne / vd / vm en passant l'évolution de consommation dans l'objet `conso`.

En session:
- le Comptable peut afficher le `compteurs` de n'importe quel compte "A" ou "O".
- les délégués d'une partition ne peuvent faire afficher les `compteurs` _que_ des comptes "O" de leur partition.

### Mutation d'un compte _autonome_ en compte _d'organisation_
Le compte passe en "O".

Le Comptable ou un délégué désigne le compte dans ses contacts et vérifie que c'est un compte "A".

Les quotas `qc / qn / qv` sont ajustés par le sponsor / comptable:
- de manière à supporter au moins le volume actuels n / v,
- en respectant les quotas de la partition courante.

L'opération de mutation:
- inscrit le compte dans la partition courante,
- dans `compteurs` du `comptas` du compte:
  - remise à zéro du total abonnement et consommation des mois antérieurs (`razma()`):
  - l'historique des compteurs et de leurs valorisations reste intact.
  - les montants du mois courant et des 17 mois antérieurs sont inchangés,
  - MAIS les deux compteurs `aboma` et `consoma` qui servent à établir les dépassements de coûts sont remis à zéro: en conséquence le compte va bénéficier d'un mois (au moins) de consommation _d'avance_.
- inscription d'un item de chat.

### Rendre _autonome_ un compte "O"
C'est une opération du Comptable et/ou d'un délégué.

L'opération de mutation:
- retire le compte de sa partition.
- comme dans le cas ci-dessus, remise à zéro des compteurs total abonnement et consommation des mois antérieurs.
- dans `comptas`:
  - `solde` vaut un pécule de 2c, le temps de générer un ticket et de l'encaisser.
  - les listes des `tickets dons` sont vides.

### Sponsoring d'un compte "O"
Rien de particulier : `compteurs` est initialisé. Sa consommation est nulle, de facto ceci lui donne une _avance_ de consommation moyenne d'au moins un mois.

### Sponsoring d'un compte "A"
`compteurs` est initialisé, sa consommation est nulle mais il bénéficie d'un _solde_ minimal pour lui laisser le temps d'enregistrer son premier crédit.

Dans `comptas` on trouve:
- un `solde` de 2 centimes.
- des listes `tickets dons` vide.

# Tâches différées (_triggers_) et périodiques

Une opération effectue dans sa transaction les mises à jour immédiates de manière à ce que la cohérence des données soit garantie. En conséquence de ces transactions il peut rester des activités d'optimisation et / ou de nettoyage à exécuter qui peuvent être _différées_.

**Une tâche périodique** a pour objectif de détecter les changements d'états liés au simple passage du temps:
- compte résilié en raison d'une non-utilisation prolongée,
- sponsorings ayant dépassé leur date limite,
- production d'états mensuels.

**_Exemple:_**
- _transaction principale_: un groupe est supprimé. Dès cet instant toute opération tentant d'agir sur le groupe sortira en exception parce que le groupe n'existe plus.
- _tâche différée_: purge des membres et des notes.

Un document `taches` enregistre toutes ces tâches différées:
- une opération _principale_ peut enregistrer des tâches à  sous contrôle transactionnel.
- un _démon_ est lancé s'il n'était pas en cours, qui va scruter `taches`, 
  - en extraire la plus ancienne en attente de traitement,
  - la traiter en une transaction, ce qui peut le cas échéant ajouter d'autres tâches,
  - in fine la retirer de `tâches`, ou pour une tâche périodique la réinscrire pour le lendemain typiquement.
  - puis rechercher la tâche suivante à exécuter et en cas d'absence se rendormir.

**Une tâche non périodique:**
- a un code opération (celle de son traitement),
- est associée à un espace: elle n'est pas candidate à l'exécution si son espace est figé ou clos.
- a un document cible identifié par id ou id / ids.
- est exécutée sous privilège administrateur et n'enregistre pas ses consommations,
- a une date-heure au plus tôt: quand une tâche a rencontré une exception, sa date-heure au plus tôt permet de laisser s'écouler un certain délai avant une nouvelle exécution.
- n'a pas de rapport de bonne exécution, la tâche étant détruite.
- elle a un rapport d'exception décrivant l'exception qui a interrompu sa dernière exécution.

**Une tâche périodique:**
- a un code opération (celle de son traitement),
- se réinscrit systématiquement pour plus tard en **début** de tâche afin de ne pas être lancée par deux démons de deux _serveurs / cloud function_ parallèle,
- est exécutée sous privilège administrateur et n'enregistre pas ses consommations.
- peut avoir un rapport d'exception décrivant l'exception qui a interrompu sa dernière exécution.

**Espace clos / figé**
- **une tâche non périodique n'est pas lancée** si son espace est clos ou figé.
- une tâche périodique teste dans son exécution l'état des espaces en fonction de chaque document qu'elle doit traiter, et ne traite que ceux dont l'espace n'est ni clos ni figé.

**Multi serveur / cloud function**: empêcher 2 serveurs de lancer la même tâche
- _début de tâche_: mise à jour de la date-heure au plus tôt. _Comme si_ la relance de la tâche était déjà planifiée.
- _fin de tâche_: purge de la tâche pour une tâche _non périodique_ et changement de l'heure de relance pour une tâche périodique.

**Articulation avec le GC**
- le GC est une pseudo-opération sollicitée de l'extérieur par une URL qui réveille le _démon_.
- en effet en l'absence de trafic, en l'absence du réveil du GC, des tâches non périodiques en exception pourraient n'être relancées que tardivement, des rapports mensuels ne pas être calculés à temps.

### L'administrateur dispose d'un accès aux tâches, et peut:
- voir la liste des tâches,
  - soit les tâches périodiques,
  - soit les tâches non périodiques _d'un espace_.
- voir le dernier compte-rendu d'exception d'une tâche.
- ajouter ou supprimer une tâche,
- réveiller le démon.

### Réveiller le _démon_ au démarrage du serveur ?
- pour un _serveur_ ça fait sens,
- pour une _cloud function_ c'est un overhead de lecture systématique de `taches`.

La table / document n'a pas de _data_ mais directement des colonnes / attributs exposés dans la base :
- `op` : code de l'opération.
- `ns` : 0 pour une tâche périodique, sinon le ns de la tâche.
- `dh` : date-heure au plus tôt d'exécution.
- `id` : id de l'objet principal concerné, '' pour un périodique.
- `ids`: identifiant secondaire ou ''.
- `exc` : rapport d'exception de la dernière exécution.
- `dhf` : date-heure de fin pour une tâche GC.
- `nb : nombre d'items traités à la dernière exécution d'une tâche GC.
`
Tâche candidate:
- la plus petite `dh` avec dh supérieure à l'instant présent.
- dont le `ns` est '' OU dans la liste des ns _ouverts_ (existants, non figé, non clos) OU n'est pas dans la liste fermée des ns _fermés_.

## Liste des tâches non périodiques

### GRM : purge des membres d'un groupe supprimé
Arguments:
- `id` : id du groupe.

### AGN : purge des notes d'un groupe ou avatar supprimé
Arguments:
- `id` : id du groupe / avatar.

### AGF : purge d'un fichier supprimé OU des fichiers attachés aux notes d'un groupe ou avatar supprimé
Arguments:
- `id` : id du groupe / avatar.
- `ids` : ids du fichier si la suppression ne concerne que ce fichier.

_Remarque_: pour la suppression _d'un_ fichier, l'opération principale,
- en phase 2, inscrit la tâche AGF pour le fichier à supprimer,
- en phase 3 essaie de supprimer le fichier du _Storage_ et en cas de succès supprime la tâche AGF inscrite en phase 2. La tâche est, dans le cas favorable, supprimée _avant_ que le démon n'ait pu être réveillé (en fin de l'opération).

### AVC : gestion et purges des chats de l'avatar
Arguments:
- `id` : id de l'avatar.

Liste les chats de l'avatar et pour chacun:
- met à jour le statut / cv du `chatE` correspondant.
- purge le `chatI`

### ESP : mise à jour des dlv des comptes d'un espace dont la date-limite fixée par l'administrateur a changé
Arguments:
- `ns` : celui de l'espace concerné
- `ids` : `dla` pour retrouver les comptes de l'espace à traiter.

Un tour d'exécution par compte.

## Liste des tâches périodiques

### DFH : détection d'une fin d'hébergement
Filtre les groupes dont la `dfh` est dépassée:
- immédiatement pour chaque groupe _suppression du groupe_.

### DLV : détection d'une résiliation de compte
Filtre les comptes dont la `dlv` est dépassée:
- immédiatement pour chaque compte _suppression du compte_.

### TRA : traitement des transferts perdus
Filtre les transferts par `dlv`:
- immédiatement pour chaque transfert, purge dans le _Storage_

### VER : purge des versions supprimées depuis longtemps

### STC : statistique "mensuelle" des comptas (avec purges)

### STT : statistique "mensuelle" des tickets (avec purges)

# Connexion et Synchronisation au fil de l'eau d'une session de l'application Web

**Principes:**
- à la fin de la phase de _connexion_, 
  - tous les documents _synchronisés_ du périmètre du compte sont en mémoire et cohérents entre eux. 
    - 1 `espaces`
    - 1 sous-arbre `comptes comptis invits`
    - N sous-arbres `avatars ... notes sponsorings chats tickets`
    - M sous-arbres `groupes ... notes membres`
  - si la session est _synchronisée_ cet état est aussi celui de la base base locale IDB qui a été mise à jour de manière cohérente pour le compte, puis pour chaque avatar, chaque groupe.
- par la suite l'opération `Sync` maintient cet état.

> La création d'un compte par acceptation de sponsoring amène la session au même point que la _connexion_ ci-dessus.

> Les trois autres documents du périmètre du compte `syntheses partitions comptas` sont chargés à la demande.

## L'objet `DataSync`
Cet objet sert:
- entre session et _service OP_ a obtenir les documents resynchronisant la session avec l'état de la base.
- dans une base locale IDB: à indiquer ce qui y est stocké et dans quelle version.

**Les états successifs _de la base_ sont toujours cohérents**: _tous_ les documents de _chaque_ périmètre d'un compte sont cohérents entre eux.

L'état courant d'une session en mémoire et le cas échéant de sa base locale IDB, est consigné dans l'objet `DataSync` ci-dessous:
- chaque sous-arbre d'un avatar ou d'un groupe est _cohérent_ (tous les documents sont synchrones sur la même version `vs`),
- en revanche il peut y avoir (plus ou moins temporairement) des sous-arbres à jour par rapport à la base et d'autres en retard.

**L'objet `DataSync`:**
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

### Synchronisation en session
Après la phase de _connexion_, l'état en mémoire est cohérent et stable, avec _écoute des web push_ activée en permanence: ces avis sont reçus par _notifications poussées au Browser_. Le service PUBSUB voit passer toutes les mises à jour et connaît les périmètres de toutes les sessions.

**Remarques:**
- les avis de mise à jour des sous-arbre _compte_, sous-arbre _avatar_, sous-arbre _groupe_ peuvent parvenir dans un ordre différent de celui dans lequel les mises à jour sont intervenues;
- un avis de mise à jour de `espaces` est décorrélé des autres: il est traité isolément dès son arrivée.
- en revanche un avis sur comptes peut parvenir après un avis sur un de ses avatars: pour éviter cette discordance, l'état de compte est toujours relu (si nécessaire) à chaque `Sync`.
- il se _POURRAIT_ qu'un sous-arbre (complet) _avatar_ soit remis à jour AVANT un sous-arbre _groupe_, dans l'ordre inverse des opérations sur le serveur. Mais cette discordance entre la vue en session et la réalité,
  - va être temporaire,
  - est fonctionnellement quasi impossible à discerner,
  - n'a pas de conséquence sur la cohérence des données.

## Connexion en mode _avion_
Phase unique:
- lecture de l'item de _boot_ de la base locale:
  - il permet d'authentifier le compte (et d'acquérir sa clé K),
  - lecture du `DataSync` de la base locale,
- mise en _mémoire tampon compilée_ depuis la base locale,
  - des documents `espaces comptes comptis invits`.
  - des documents des sous-arbres _avatar_ et _compte_.
- **mise à jour des _stores_ des documents compilés** en une séquence sans interruption (sans `await`) afin que la vision graphique soit cohérente.

## Synchronisation au fil de l'eau
Au fil de l'eau il parvient des notifications de mises à jour de _versions_. 

Une table _queue de traitements_ mémorise pour chaque sous-arbre, son `id` et la version notifiée par l'avis de mise à jour. Elle regroupe ainsi des événements survenus très proches.

Les avis de mises à jour dont la version est inférieure ou égale à la version déjà détenue dans les _stores_ de la session, sont ignorés.

### Itération pour vider cette queue
Tant qu'il reste des traitements à effectuer, une opération `Sync` est soumise:
- le `DataSync` est celui courant,
- L'id du sous-arbre est:
  - `id` si l'avis de changement concerne le sous-arbre _compte_.
  - `ida`, l'id du sous-arbre si la notification correspond à un sous-arbre.

Le traitement standard de retour,
- met à jour la base locale en une transaction,
- met à jour les _store_ de la session sans interruption (sans `await`).

# Annexe I: déclaration des index

## SQL
`sqlite/schema.sql` donne les ordres SQL de création des tables et des index associés.

Rien de particulier : sont indexées les colonnes requérant un filtrage ou un accès direct par la valeur de la colonne.

## Firestore #A REVOIR
`firestore.index.json` donne le schéma des index: le désagrément est que pour tous les attributs il faut indiquer s'il y a ou non un index et de quel type, y compris pour ceux pour lesquels il n'y en a pas.

**Les règles génériques** suivantes ont été appliquées:

_data_ n'est jamais indexé.

Tous les attributs apparaissant dans une _query_ avec un _where_ donne lieu à un index, voire un index composite (id / v ...).

Pour `sponsorings` `ids` sert de clé d'accès direct et a donc un index **collection_group**, pour les autres l'index est simple.

Autres index:
- `hXR` sur `comptas`: accès à la connexion par phrase secrète.
- `hYR` sur `avatars`: accès direct par la phrase de contact.
- `dfh` sur `groupes`: détection par le GC des groupes sans hébergement.
