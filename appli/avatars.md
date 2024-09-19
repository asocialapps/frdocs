---
layout: page
title: Créer, gérer et supprimer ses avatars
---

Un compte a toujours un avatar principal et ne peut pas en changer.

Au cours de sa vie un compte peut,
- créer des avatars secondaires en leur donnant un nom.
- les supprimer, ce qui est une opération qui ne se lance pas à la légère.

> L'intérêt d'avoir plusieurs avatars est de compartimenter sa vie : dans le cas de partage de notes dans des groupes, le titulaire du compte choisit de présenter l'un ou l'autre de ses avatars en fonction du thème traité dans le groupe.

## Création d'un avatar
Pour un compte la création se limite à fournir un _nom_, celui qui figure en première ligne de sa _carte de visite_.

La création d'un avatar lui attache les données immuables suivantes:
- la **clé** de cryptage aléatoire de sa _carte de visite_ (qui est mémorisée cryptée par la clé K du compte).
- son **identifiant** à 12 lettres et chiffres (dérivé de la clé).
- une paire de clés RSA, dont la clé _privée_ est mémorisée cryptée par la clé K du compte.

### Carte de visite d'un avatar

La **carte de visite** d'un avatar est modifiable par le compte et comporte :
- une photo facultative de petite dimension,
- un court texte incluant le nom de l'avatar: par exemple complétant le nom `Charles` par `Charles, Roi des esturgeons et d’Écosse`. 

**Le début de la première ligne de texte sert de _nom_ à l'affichage**, en général complété par les 4 dernières lettres / chiffres de l'identifiant..

La _carte de visite_ est mémorisée cryptée par la clé de l'avatar et n'est lisible que par les _contacts_ de l'avatar.

### Phrase de contact d'un avatar
Un avatar peut se déclarer une _phrase de contact_, par exemple `Superman n'aime pas les choux fleurs`. Les 12 premiers caractères de cette phrase ne doivent pas être ceux d'une phrase actuellement enregistrée.

Il est conseillé de n'enregistrer une telle phrase que pour un objectif précis et de l'effacer ensuite au plus tôt.

#### Usage 1 : mise en contact d'avatars ne se connaissant pas directement
La _phrase de contact_ permet à un avatar A qui n'a pas connaissance de la _carte de visite_ de B de permettre à B d'ouvrir un chat avec lui A, A et B devenant après cela des contacts permanents.
- le titulaire du compte A peut communiquer cette phrase à B rencontré dans le vrai monde mais qu'il n'a pas en _contact_ personnel. 
- il peut aussi la communiquer dans un chat avec un contact M qui a pour B dans ses contacts pour que M la communique à B. En l'occurrence M joue un rôle de _médiateur_ entre A et B pour les mettre en contact, sans pour autant avoir un groupe en commun.

La phrase DOIT être détruite dès son office rempli afin que personne d'autre ne puisse l'utiliser pour ouvrir un chat non sollicité.

#### Usage 2 : donner son accord pour muter de compte A en O ou O en A
Muter de type de compte _Autonome_ / _Organisation_ est une opération qui a des conséquences pour l'organisation: seuls le Comptable ou un de ses _délégués_ peuvent l'exécuter.

Mais ceci suppose l'accord du compte à muter: le Comptable ou son délégué doit citer la _phrase de contact_ du compte à muter valant accord, par exemple `Bon pour accord de mutation: signé Jules l'intrépide`.

## Suppression d'un avatar
Elle peut être demandée par le titulaire du compte et peut avoir des conséquences importantes:
- perte définitive des notes personnelles de l'avatar.
- tous les chats de l'avatar sont marqués pour les interlocuteurs comme s'adressant à un avatar _disparu_: les textes sont désormais non modifiables.
- les participations aux groupes sont supprimées:
  - si l'avatar était _hébergeur_ du groupe, celui-ci se retrouve contraint à décroître en volume et à devoir être remplacé en tant qu'hébergeur sous peine de disparition du groupe au bout de 3 mois.
  - si l'avatar était le dernier _actif_, le groupe disparaît, ces invitations éventuellement en cours étant annulées.

L'application affiche ces conséquences à l'écran et attend une validation de chacun de ces points, puis une validation globale, afin de procéder à cette action qui ne supporte aucun remord.

> **La suppression du dernier avatar d'un compte** (son avatar principal) procède de la même façon et **supprime le compte**.
