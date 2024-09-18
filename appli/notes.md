---
layout: page
title: Notes personnelles
---

**Une note porte un texte** d'au plus 5000 caractères pouvant s'afficher avec un minimum de _décoration_, gras, italique, titres, listes ... Ce texte est modifiable.

**Des fichiers peuvent être attachés à une note**
- les types de fichiers les plus usuels (`.jpg .mp3 .mp4 .pdf ...`) s'affichent directement dans le navigateur, les autres sont téléchargeables.
- il est possible d'ajouter et de supprimer des fichiers attachés à une note.
- quand plusieurs fichiers portant le même _nom_ dans la note, ils sont vus comme des _révisions_ successives, qu'on peut garder, ou ne garder que la dernière, ou celles de son choix.

### Vue hiérarchique: note _parent_ d'un note
- les notes apparaissent à l'écran sous forme hiérarchique, une note _parent_ ayant en dessous d'elle des notes _enfants_ (ou aucune).
- les notes n'ayant pas de note _parent_ apparaissent rattachées à celui des avatars du compte a qui elle appartient: cet avatar est une _racine_ de la hiérarchie des notes.

> Un avatar peut créer des notes **personnelles**, les mettre à jour, les supprimer et **les indexer par des mots clés personnels**. Elles sont cryptées, comme toutes les données des comptes, et seul le titulaire du compte a, par l'intermédiaire de sa phrase secrète, la clé de cryptage apte à les décrypter.

TODO: révisions, fichiers en mode avion

#### Fichiers attachés à un secret
Pour chaque fichier il est enregistré :
- son numéro identifiant aléatoire.
- son **nom** : comme un _nom de fichier_, d'où un certain nombre de caractères _interdits_ dont le `/`. Plusieurs fichiers attachés à un même secret peuvent porter le même nom, ce sont des _versions_ pour un nom donné.
- son **a propos**, texte facultatif d'au plus 250 caractères utile en particulier pour expliciter le nom, donner un _titre_ ou un commentaire qualifiant une _version_ lorsqu'il existe plusieurs fichiers de même nom.
- sa **date-heure d'enregistrement** dans l'application (pas celles de création ou de dernière modification de son fichier d'origine).
- son **type MIME** permettant de savoir si c'est une photo, un clip vidéo, un PDF ... Beaucoup de types de fichiers s'affichent directement dans les navigateurs.
- sa **taille** originale en octets.
- le **digest SHA-256** de son texte d'origine, permettant de s'assurer si nécessaire de sa non déformation entre sa source primitive et ses restitutions.

**Les fichiers sont stockés dans un espace dédié sur un serveur de fichiers**, ils sont compressés si c'est un fichier de type `text/...`, puis cryptés dans l'application avant d'être envoyés sur le réseau, et décryptés dans l'application après téléchargement depuis le serveur de fichier.

>On ne peut **qu'ajouter ou supprimer** des fichiers. Quand on ajoute un fichier, il est proposé de supprimer simultanément un ou plusieurs autres fichiers de même nom : ceci donne l'impression d'une mise à jour qui permettrait de plus d'en gérer les _versions_ tout en évitant l'inflation du volume.

## Téléchargement d'une sélection de secrets

Une opération de téléchargement permet d'écrire en clair sur un disque local une sélection de secrets, **leurs textes et leurs fichiers**. Ceci requiert,
- un PC Linux ou Windows,
- le téléchargement et l'installation d'un petit utilitaire à lancer avant le d'effectuer le téléchargement et à arrêter ensuite par sécurité. En effet un navigateur n'a pas le droit d'écrire sur l'espace de fichiers de l'appareil, ce que l'utilitaire peut faire.

>Les téléchargements peuvent être coûteux en transfert sur le réseau et les hébergeurs du serveur de fichiers peuvent les facturer et / ou les limiter. Ils sont **ralentis** par des temporisations dès que la moyenne des volumes téléchargés sur les 14 derniers jours dépasse le quota V2 d'un compte (d'autant plus que le dépassement est important).
