---
layout: page
title: Différences de fond entre les réseaux sociaux et AsocialApp
---

Elles sont importantes : **AsocialApp** donne accès à **des** espaces **a-sociaux**, privés et cryptés de bout en bout, pour des comptes _numérotés et anonymes_, bref pas _sociaux_ du tout.

Clairement **AsocialApp** n'est pas _monétisable_ par la publicité ciblée, donc d'une manière ou d'une autre payante, aucune information personnelle / intelligible ne pouvant être captée.

### Public versus privé
Un _réseau social_ est public, ouvert au plus grand monde, parfois avec des millions d'utilisateurs ayant une identité visible (un nom / pseudo, une photo, etc.). L'objectif du fournisseur est de capturer le maximum d'informations sur les personnes ayant un compte afin de les monétiser. D'une manière ou d'une autre **les comptes sont reliés à un identifiant du monde réel, adresse e-mail, numéro de téléphone, ...**

A l'inverse chaque _organisation_ peut avoir son propre _espace **AsocialApp** ayant son URL spécifique_  réservé à ses membres: en centaines, voire en milliers, certainement pas des millions.
- **les comptes y sont identifiés par un code aléatoire sans signification**, sans aucune relation avec des identifiants du monde réel.
- _l'identité_, la _carte de visite_ de l'avatar d'un compte n'est visible que par les quelques contacts qu'il a agréé.
- l'accès à son compte est subordonné à la connaissance par son titulaire de sa phrase secrète (d'au moins 24 signes) qu'il est seul à connaître et n'est écrite nulle part dans l'application.

### Inscription libre versus cooptation
L'inscription à un _réseau social_ est, en général, libre, ouverte à tous.

A l'opposé pour créer un compte d'une organisation ayant un espace **AsocialApp**, son titulaire doit être _sponsorisé_ par un compte existant.

Dans **AsocialApp** il n'existe aucun répertoire nominatif des comptes et avatars: 
- il n'est possible de joindre que ses propres _contacts_,
- toutes les identités humainement interprétables sont **cryptées**, ne sont jamais disponibles en clair, ni sur le serveur central, ni sur le réseau.

### Groupes ouverts versus groupes sur invitation
Dans un _réseau social_ il est en général possible de créer des _groupes_ d'utilisateurs.

Avec **AsocialApp** c'est aussi le cas mais un membre ne peut proposer à l'invitation qu'un de ses _contacts_: il est impossible d'obtenir un contact par son nom / pseudo ou tout autre critère de recherche, les données d'identification n'étant pas disponibles sur les serveurs.

### Textes publics versus textes cryptées
Dans **AsocialApp** tous les textes / fichiers humainement interprétables sont cryptés de bout en bout, le serveur (ni le réseau) n'y ont accès en clair.

Les cryptage / décryptage sont effectués à partir de clés générées et cryptées elles-mêmes par la _phrase secrète_ déclarée par le compte. C'est le cas pour:
- les données de la _carte de visite_ de chaque avatar d'un compte,
- les textes des _chats_ entre deux comptes où à l'intérieur d'un groupe,
- les textes des _notes_ personnelles ou d'un groupe, ainsi que leurs fichiers attachés dont l'espace de stockage ne connaît jamais que des contenus cryptés qui ne peuvent être interprétés que dans les sessions des comptes en ayant la clé associée.

> Les données dans la _base de données_ sont également cryptées, son hébergeur technique n'a pas accès aux _méta-données_ de liens entre les comptes / les groupes / les chats.

### Fournisseur unique versus fournisseurs multiples
Un _réseau social_ est _son propre fournisseur_, il a installé une application et ses serveurs depuis un ensemble de logiciels privés (en général).

**AsocialApp** n'est QU'UN LOGICIEL, disponible en _open-source_. N'importe qui peut déployer l'application sur ses serveurs, avec,
- ses propres _Conditions Générales de Vente_, tarifaires et éthiques,
- ses URLs d'accès à ses services,
- sa propre équipe d'administration inconnue des autres fournisseurs.

> En conséquence quand une _organisation_ souhaite utiliser un espace **AsocialApp**, elle doit trouver un fournisseur qui a déployé le logiciel et qui accepte d'héberger un espace pour elle.
 
> L'organisation peut aussi déployer elle-même l'application et être son propre fournisseur, ce qui requiert un minimum d'expérience à ce genre d'exercice (et un peu de temps).
