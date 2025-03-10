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
- une note n'ayant pas de note _parent_ apparaît rattachée à celui des avatars du compte (ou du groupe) a qui elle appartient: cet avatar est une _racine_ de la hiérarchie des notes.

> Un avatar peut créer des notes **personnelles**, les mettre à jour, les supprimer et **les indexer par des mots clés personnels**. Elles sont cryptées, comme toutes les données des comptes, et seul le titulaire du compte a, par l'intermédiaire de sa phrase secrète, la clé de cryptage apte à les décrypter.

### Informations à propos d'un fichiers attaché à une note
Pour chaque fichier il est enregistré :
- son identifiant généré.
- son **nom** : comme un _nom de fichier_, d'où un certain nombre de caractères _interdits_ dont le `/`. Plusieurs fichiers attachés à une même note peuvent porter le même nom, ce sont des _révisions_ pour un nom donné.
- son **a propos**, texte facultatif utile en particulier pour qualifier une _révision_ lorsqu'il existe plusieurs fichiers de même nom.
- sa **date-heure d'enregistrement** dans l'application.
- son **type MIME** permettant de savoir si c'est une photo, un clip vidéo, un PDF ... Beaucoup de types de fichiers s'affichent directement dans les navigateurs.
- sa **taille** originale en octets.
- le **digest SHA-256** de son texte d'origine, permettant de s'assurer si nécessaire de sa non déformation entre sa source primitive et ses restitutions.

**Les fichiers sont stockés dans un _storage_ dédié à cet usage**:
- ils sont compressés quand ce sont des fichiers de type `text/...`, 
- ils sont cryptés dans l'application avant d'être envoyés sur le réseau, et décryptés dans l'application après téléchargement depuis le serveur de fichier.

> On ne peut **qu'ajouter ou supprimer** des fichiers. Quand on ajoute un fichier, il est proposé de supprimer simultanément un ou plusieurs autres fichiers de même nom : ceci donne _l'impression_ d'une mise à jour mais permet d'en gérer les _révisions_ (donc un possible retour en arrière) tout en évitant l'inflation du volume.

## Téléchargement local d'une sélection de notes
L'application Web peut afficher une sélection de notes selon plusieurs critères de filtrage, éventuellement aucun.

Une action _locale au poste_ de téléchargement permet d'écrire en clair sur un disque local la sélection affichée, **leurs textes et optionnellement leurs fichiers qui sont alors obtenus du serveur**. Ceci requiert,
- un PC Linux ou Windows,
- l'utilisation d'un petit utilitaire (téléchargeable et sans installation) à lancer avant d'effectuer le téléchargement et à arrêter ensuite par sécurité.

> Un navigateur n'a pas le droit d'écrire sur l'espace de fichiers de l'appareil, ce que l'utilitaire peut faire.

> Les téléchargements peuvent être coûteux en transfert sur le réseau et les hébergeurs du serveur de fichiers peuvent les facturer et / ou les limiter. Ils peuvent être **ralentis** par des temporisations si la consommation de calcul (qui inclut le volume téléchargé) est hors des limites des quotas du compte.

## Fichiers accessibles en mode _avion_ et usage du _presse-papier_

Lire les explications dans la page suivante:

[Modes de connexion synchronisé, avion, incognito](./modessync.html)
- Les modes de connexion, fichiers accessibles en mode _avion_, le _presse-papier_.
