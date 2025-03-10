---
layout: page
title: Les comptes et leurs avatars
---

La création de comptes se fait par _cooptation_ : un compte _sponsor_ prépare une fiche de sponsoring à destination d'un futur compte. Si ce dernier en accepte les termes, le compte sera ouvert.

> Pour information: par défaut à la création de l'espace d'une organisation, les comptes _autonomes_ n'y sont pas autorisés afin que le Comptable ait un contrôle initial complet. Une commande simple lui permet d'autoriser la création de comptes _autonomes_.

> L'ensemble des ressources, espaces et calcul, peut être découpé en _partition_ afin d'en déléguer la distribution des ressources entre les comptes "O", chacun appartenant à _sa_ partition.

### Qui peut être _sponsor_ ?
- Le Comptable peut sponsoriser des comptes "A" et des comptes "O".
- Un compte "A" ne peut sponsoriser que des comptes "A",
- un compte "O" que le Comptable à intronisé _délégué de sa partition_ peut sponsoriser,
  - des comptes "O" dans sa partition, en les intronisant ou non, eux-mêmes _délégué_ de la partition.
  - des comptes "A".

### La fiche de _sponsoring_
- Elle est identifiée par une phrase de sponsoring que le sponsor communique à son sponsorisé. 
  - elle fait au moins 24 signes, les 12 premiers devant être uniques.
- Elle fixe un premier nom pour le sponsorisé que ce dernier pourra changer ultérieurement.
- Le sponsor peut faire un _don_ au sponsorisé, pour autant que le solde du sponsor le permette. Quand le sponsorisé est un compte "A", ce don lui permet de commencer à utiliser son compte tout de suite sans attendre un premier versement.
- Le sponsor attribue au sponsorisé trois _quotas_:
  - un quota QN de _nombre de documents_ (nombre de notes + nombre de chats + nombre de participations actives aux groupes), 
  - un quota QV de _volume de fichiers_,
  - et un quota QC de calcul mensuel. 
- Ces quotas sont prélevés sur les quotas inutilisés de la partition pour un compte "O" ou sur le quota global de l'espace réservé aux comptes "A".

La fiche de sponsoring est complétée par un _mot de bienvenu_ du sponsor au sponsorisé:
- habituellement ce mot initialise un _chat_ entre sponsor et sponsorisé, qui seront ainsi mutuellement _contacts_ et connaîtront leurs cartes de visite.
- toutefois, le sponsor peut refuser que ce _chat_ (et ce contact) s'établisse.

La _fiche de sponsoring_ a une durée de validité de 30 jours. Si elle n'a pas été acceptée / refusée dans ce délai elle s'autodétruit.

### Création du compte _sponsorisé_
Le futur titulaire accède à sa fiche de sponsoring en citant la phrase que le sponsor lui a communiqué:
- il peut revoir les conditions qui s'y trouvent, la carte de visite du sponsor, les quotas attribués, le mot de bienvenu.
- s'il n'est pas satisfait de ces conditions, il peut refuser le _sponsoring_ en inscrivant par politesse un mot d'explication.
- s'il est satisfait, le sponsorisé:
  - choisit sa phrase secrète qu'il saisit deux fois,
  - remplit un mot de remerciement pour le sponsor,
  - peut ne pas souhaiter ouvrir un _chat_ avec le sponsor et rester indépendant de lui, sinon il accepte et _sponsor / sponsorisé_ seront des contacts.

La _phrase de sponsoring_ est obsolète au plus 30 jours après création de la fiche de sponsoring. 

Le sponsor peut:
- lister ses sponsorings des 30 derniers jours (en particulier pour vérifier la phrase de sponsoring).
- il peut supprimer ceux qui n'ont été ni acceptés, ni refusés.

#### La _phrase secrète_
- Ses 12 premiers signes doivent être uniques dans l'espace de l'organisation.
- les 12 suivants ou plus permettent d'authentifier le compte.

> Remarque: il n'y a pas d'inconvénient à donner un texte personnel (prénoms, nom, email ...) en première partie de la phrase secrète, celle-ci est cryptée et illisible par quiconque autre que le compte lui-même.

> Tant qu'à faire, la fin de la phrase a intérêt a n'avoir aucun rapport avec son début. Sa protection vient de sa longueur.

Une phrase secrète peut être changée, à condition de connaître l'actuelle, mais il n'y a aucune obligation à le faire. C'est opportun seulement si on pense que quelqu'un vous a vu la frapper ou que l'appareil qu'on utilise a pu être _hacké_ et les frappes au clavier interceptés.

> Pour les paranos, on peut saisir sa phrase depuis un clavier visuel qui évite le problème du clavier piraté.

> Ceux qui pensent que leur appareil est bien protégé par leur mot de passe (ou tout autre procédé) ET qui n'ont pas envie de saisir au moins 24 caractères de phrase secrète, _peuvent_ déclarer des _codes PIN_ dans leur browser: voir en annexe de cette page.

# Après création du compte

Le _sponsorisé_ a un compte opérationnel et il peut y effectuer des opérations courantes.

[Créer, gérer et supprimer ses avatars](./avatars.html)

## Exemples d'opérations usuelles au démarrage

### Changer sa _carte de visite_
Typiquement y mettre une photo et plus d'information à propos de soi-même que le simple nom donné par le _sponsor_.

### Accéder à sa _liste de contacts_
A cet instant elle a (au plus) **un** contact, son _sponsor_, sauf si l'un des deux sponsor / sponsorisé n'ont pas voulu de ce contact.
- il est possible / souhaitable d'attacher à son sponsor un _commentaire_ qui rappelle au moins comment il s'appelait au moment du sponsoring: le sponsor peut en effet changer sa carte de visite quand il veut. En attribuant un nom / commentaire personnel le sponsor pourra être retrouvé par filtrage sur le contenu du commentaire personnel attaché au sponsor.
- il est possible de lui attacher des _hashtags_, qui facilitent les filtrages par _catégorie_ de contact.
- on peut aussi ouvrir le _chat_ d'un contact: on y voit pour l'instant le mot de bienvenu du sponsor et le mot de remerciement du sponsorisé.

#### _Hashtags_
Un _hashtag_ est une suite courte de lettres minuscules non accentuées et de chiffres, sans espaces.
- la boîte de dialogue de saisie propose tous ceux déjà utilisés par le compte,
- il suffit d'en saisir un nouveau si aucun ne convient.
- quand un _hashtag_ n'est plus utilisé nulle part dans le compte, il disparaît de la liste.

## Compte "A": ajustement de ses quotas
Son abonnement à la création correspond à un minimum de _documents_, de _volume de fichiers attachés_ et de _ressource de calcul_: il est opportun d'y mettre des valeurs réalistes.
- le quota QN du nombre de documents est un entier dont l'unité correspond à 100 documents.
- le quota QV du volume de fichiers est un entier dont l'unité correspond à 100MB.
- le quota QC est en unités monétaires (en _centimes_). Chaque lecture / écriture / transfert de fichier a un coût en centimes.

[Abonnement, consommation, tarifs](/appli/tarifs.html)

## Compte "O" _délégué_: ajustements des quotas des comptes de sa partition
Un compte _délégué_ peut ajuster les quotas des comptes "O" de sa partition, et en conséquence les siens.

[Gestion des comptes d'une "partition" par ses _délégués_ et le Comptable](./partitions.html)

## Enregistrement de crédits
Afin de préserver l'anonymat dans le _vrai_ monde des comptes, la procédure de _paiement_ et d'enregistrement des crédits s'effectue de la manière suivante.

Le compte, 
- ouvre la fenêtre des paiements et déclare un paiement avec son montant. Il en résulte un _numéro de ticket de paiement_ et un paiement enregistré en attente.
- il fait parvenir au Comptable par le moyen prévu par l'organisation (typiquement un virement d'une banque, mais aussi tout autre procédé accepté par l'organisation) le montant correspondant accompagné du numéro du ticket obtenu ci-avant.

Quand le Comptable reçoit un paiement avec un numéro qui l’anonymise, il enregistre le montant reçu pour le ticket (qu'il trouve dans la liste des tickets en attente).
- si le compte était _en rouge_ pour solde négatif, dès l'enregistrement par le Comptable, l'indicateur s'efface si le crédit est suffisant.
- les tickets enregistrés par le Comptable restent visibles pour lui jusqu'à M+2 de leur création.

> Les _tickets_ étant enregistrés cryptés par la clé des comptes, aucune corrélation ne peut être faite entre la source d'un _paiement_ et le compte qui en bénéficie.

## A sa création une organisation _n'accepte pas_ de comptes _autonomes_
- Le Comptable peut lever cette interdiction et en autoriser la création,
- il peut aussi supprimer cette autorisation: cela n'a aucun effet sur les comptes _autonomes_ existants et ne vaut que pour les créations / mutations ultérieures.

## Changement de la phrase secrète
Un compte peut changer sa phrase secrète:
- il doit se connecter avec la phrase actuelle,
- il donne (deux fois) sa nouvelle phrase dont les 12 premiers signes ne doivent pas être ceux de la phrase secrète d'un autre compte.

> **Remarque**: quand on se connecte depuis un navigateur en mode _avion_ il faut donner la phrase secrète qui était en cours lors de la dernière connexion en mode synchronisé sur ce navigateur.

# Pour information: la création du _Comptable_
A l'initialisation de l'espace d'une organisation, l'administrateur technique déclare une _phrase de sponsoring_ pour le Comptable de l'espace.
- tant que le Comptable n'a pas créé son compte, l'espace n'est pas utilisable.
- le Comptable _accepte_ son sponsoring et déclare sa phrase secrète.

Étant le seul compte, il est à ce moment le seul à pouvoir sponsoriser des comptes.

# Auto résiliation d'un compte
Un compte peut s'auto-résilier : tous ses avatars secondaires doivent avoir été supprimés.
- Si c'est un compte "O" ses quotas d'abonnement / consommation sont rendus à sa partition.
- Toutes ses données sont effacées, sauf ...
  - _ses notes de groupe_: dans un groupe une note _appartient_ au groupe et reste normalement accessible aux autres membres.
  - pour ses _chats_ personnels: son exemplaire disparaît mais l'exemplaire de son contact reste accessible, quoique sans mise à jour. Si le contact déclare le chat _passif / indésirable_, le contenu du _chat_ disparaît.

# Disparition d'un compte

**Un compte qui ne s'est pas connecté pendant 12 mois est déclaré *disparu*** : sa connexion est impossible et ses données seront physiquement détruites afin de ne pas encombrer inutilement l'espace (comme si le compte s'était auto-résilié). 

Comme rien ne raccorde un compte au monde réel, ni adresse e-mail, ni numéro de téléphone ... il n'est pas possible d'informer quiconque de la disparition prochaine de son compte.

> Un compte en _ACCÈS RESTREINT_, typiquement parce que son solde est débiteur, s'auto-détruit 6 mois après la date de son passage en _ACCÈS RESTREINT_.

# Mutation de comptes "A" en "O" et réciproquement

Voir cette section [Mutations de compte "O" en "A" et de "A" en "O"](./mutations__oa.html)

# Annexe: se connecter avec un code PIN
Une phrase secrète doit avoir au moins 24 signes afin de rendre sa découverte par _force brute_ (en essayant toutes les combinaisons possibles) quasiment impossible.

Mais saisir au jour le jour 24 signes ou plus depuis un appareil personnel, PC ou mobile, dont le login est déjà protégé par un mot de passe, peut se révéler fastidieux. Après tout les browsers ne rechignent pas à afficher vos mots de passe enregistrés pour autant que vous ayez réussi à vos connecter avec votre login de l'OS.

Il est possible d'utiliser un **code PIN** de quelques caractères comme alias d'une phrase secrète, avec quelques précautions:
- être certain que la connexion à votre appareil est effectivement bien protégée par un mot de passe correct,
- ne pas afficher ses codes PINs nulle part.

### Déclarer un code PIN
Supposons une phrase secrète `lessanglotslongsdelautomne` et que le code PIN `1515moi` ait été choisi comme _raccourci_ (au moins 4 signes).

Lors d'une connexion sur l'appareil choisi, saisir comme phrase secrète:

    &1515moi&lessanglotslongsdelautomne

`&` le code PIN `&` la phrase secrète

Désormais pour se connecter avec cette phrase il suffira de frapper `&1515moi&`.

Si la phrase secrète change, redéclarer le code PIN (en le changeant ou non).

### Remarques
Les correspondances entre un code PIN et leur phrase secrète sont stockées,
- dans le browser,
- cryptées par des dérivés des codes PINs,
- relativement au nom de domaine de l'application.

Il n'est pas possible de lire, même en debug, ni les valeurs des phrase secrète, ni les codes PINs mémorisés.

Les codes PINs sont spécifiques d'un appareil, ne les quittent pas, ne sont pas échangés sur le réseau: le cas échéant les réenregistrer sur d'autres appareils.

Ne pas enregistrer de code PINs sur des appareils qui ne sont pas strictement personnels ou ayant des logins _génériques / semi-publics_.

> Pour un pirate, il est évident que chercher à trouver par force brute un code PIN de 6 caractères est beaucoup plus facile que de retrouver une phrase secrète de 24 caractères (ce qui est quasi infaisable). Il aura toutefois fallu préalablement au pirate: a) dérober votre appareil, b) avoir cassé votre login (de connexion à l'OS), c) avoir écrit un logiciel spécifique de test des combinaisons.
