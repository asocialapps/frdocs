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
- un compte "O" que le Comptable à introniser _délégué de sa partition_ peut sponsoriser,
  - des comptes "O" dans sa partition, en les intronisant ou non, eux-mêmes _délégué_ de la partition.
  - des comptes "A".

### La fiche de _sponsoring_
- Elle est identifiée par une phrase de sponsoring que sponsor communique à son sponsorisé. 
  - elle fait au moins 24 signes, les 12 premiers devant être uniques.
- Elle fixe un premier nom pour le sponsorisé que ce dernier pourra changer ultérieurement.
- Si le sponsorisé est un compte "A", il doit pouvoir commencer à utiliser son compte tout de suite:
  - si le sponsor est un compte "A", il fait un don _modique_ au sponsorisé: ce don est prélevé sur son solde, mais le sponsorisé pourra le lui rendre plus tard.
  - si le sponsor est un compte "O", le sponsorisé reçoit de l'organisation un don de démarrage de 2c.
- Si le sponsorisé est un compte "O", le délégué ou le Comptable sponsor, lui attribue un quota de _documents_, de _volume de fichiers_ et de quota de calcul mensuel. Ces quotas sont prélevés sur les quotas inutilisés de la partition (ils pourront être ajustés plus tard).

La fiche de sponsoring est complétée par un _mot de bienvenu_ du sponsor au sponsorisé:
- habituellement ce mot initialisera un _chat_ entre sponsor et sponsorisé, qui seront ainsi des _contacts_ et connaîtront leurs cartes de visite mutuelles.
- toutefois, le sponsor peut refuser que ce _chat- s'établisse.

La _fiche de sponsoring_ a une durée de validité de 30 jours. Si elle n'a pas été acceptée / refusée dans ce délai elle s'autodétruit.

### Création du compte _sponsorisé_
Le futur titulaire accède à sa fiche de sponsoring en citant la phrase que le sponsor lui a communiqué:
- il peut revoir les conditions qui s'y trouve, la carte de visite du sponsor, les quotas attribués, le mot de bienvenu.
- s'il n'est pas satisfait de ces conditions, il peut refuser le _sponsoring_ en inscrivant par politesse un mot d'explication.
- s'il est satisfait, le sponsorisé:
  - choisit sa phrase secrète qu'il saisit deux fois,
  - remplit un mot de remerciement pour le sponsor,
  - peut ne pas souhaiter ouvrir un chat avec le sponsor et rester indépendant de lui, sinon il accepte et _sponsor / sponsorisé_ seront des contacts.

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

# Après création du compte

Le _sponsorisé_ a un compte opérationnel et il peut y effectuer des opérations courantes.

[Créer, gérer et supprimer ses avatars](./avatars.html)

## Exemples d'opérations usuelles au démarrage

### Changer sa _carte de visite_
Typiquement y mettre une photo et plus d'information à propos de soi-m^me que le simple nom donné par le _sponsor_.

### Accéder à sa _liste de contacts_
A cet instant elle a **un** contact, son _sponsor_, sauf si l'un des deux sponsor / sponsorisé n'ont pas voulu de ce contact.
- il est possible / souhaitable d'attacher donner à son sponsor un _commentaire_ qui rappelle au moins comment il s'appelait au moment du sponsoring: le sponsor peut en effet changer sa carte de visite quand il veut. En attribuant un nom / commentaire personnel le sponsor pourra être retrouvé par filtrage sur le contenu du commentaire personnel attaché au sponsor.
- il est possible de lui attacher des _hashtags_, qui facilitent les filtrages par _catégorie_ de contact.
- on peut aussi ouvrir le _chat_ d'un contact: on y voir pour l'instant le mot de bienvenu du sponsor et le mot de remerciement du sponsorisé.

#### _Hashtags_
Un _hashtag_ est une suite courte de lettres minuscules non accentuées et de chiffres, sans espaces.
- la boîte de dialogue de saisie propose tous ceux déjà utilisés par le compte,
- il suffit d'en saisir un nouveau si aucun ne convient.
- quand un _hashtag_ n'est plus utilisé nulle part dans le compte, il disparaît de la liste.

### Pour un compte "A", ajuster son abonnement
Son abonnement à la création correspond à un minimum de _documents_ et de _volume de fichiers attachés_: il est opportun d'y mettre des valeurs réalistes.
- le quota du nombre de documents est un entier de 0 à 250: l'unité correspond à 250 documents.
- le quota du volume de fichiers est un entier de 0 à 250: l'unité correspond à 100MB.

[Abonnement, consommation, tarifs](/appli/tarifs.html)

### Pour un compte "O" _délégué
Un compte _délégué peut ajuster les quotas des comptes "O" de sa partition, et en conséquence les siens.

Pour un compte"O" son quota comporte, en plus dues quotas nombre de documents et volume des fichiers, le coût calcul mensuel maximal décompté avec les unités suivantes:
- 1 million de lectures,
- 1 million d'écritures,
- 1GB de volume montant de fichiers,
- 1GB de volume descendant de fichiers.

Les prix unitaires de ces unités est fixé par le prestataire: il en résulte un montant en _unité monétaire_ c (un centime).

[Gestion des comptes d'une "partition" par ses _délégués_ et le Comptable](./partitions.html)

### Pour un compte "A", effectuer un paiement
Le compte est en général, _peu_ approvisionné. 
- on ouvre la fenêtre des paiements et on déclare un paiement avec son montant.
- il en résulte un _numéro de ticket de paiement_ et un paiement en attente.
- on fait parvenir au Comptable par le moyen prévu par l'organisation le montant correspondant accompagné du numéro du ticket obtenu ci-avant.

Quand le Comptable recevra un paiement avec un numéro qui l’anonymise, il enregistre le montant reçu pour le ticket (qu'il trouve dans la liste des tickets en attente).
- si le compte était _en rouge_ pour insuffisance de crédit, dès l'enregistrement par le Comptable, l'indicateur s'efface. 

# Pour information: la création du _Comptable_
A l'initialisation de l'espace d'une organisation, l'administrateur technique déclare une _phrase de sponsoring_ pour le Comptable de l'espace.
- tant que le Comptable n'a pas créé son compte, l'espace n'est pas utilisable.
- le Comptable _accepte_ son sponsoring et déclare sa phrase secrète.

Étant le seul compte, il est à ce moment le seul à pouvoir sponsorisé des comptes.

# Auto résiliation d'un compte
Un compte peut s'auto-résilier : tous ses avatars secondaires doivent avoir été supprimés.
- Si c'est un compte "O" ses quotas d'abonnement / consommation sont rendus à sa partition.
- Toutes ses données sont effacées, sauf ...
  - _ses notes de groupe_: dans un groupe une note _appartient_ au groupe et reste normalement accessible aux autres membres.
  - pour ses chats personnels: son exemplaire disparaît mais l'exemplaire de son contact reste accessible, quoique sans mise à jour. Si le contact déclare le chat _indésirable_, le contenu du chat disparaît.

# Disparition d'un compte

**Un compte qui ne s'est pas connecté pendant 12 mois est déclaré *disparu*** : sa connexion est impossible et ses données seront physiquement détruites afin de ne pas encombrer inutilement l'espace (comme si le compte s'était auto-résilié). 

Comme rien ne raccorde un compte au monde réel, ni adresse e-mail, ni numéro de téléphone ... il n'est pas possible d'informer quiconque de la disparition prochaine de son compte.
