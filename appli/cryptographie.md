---
layout: page
title: Quelques éléments de cryptographie
---

## La cryptographie pour les nuls
### Cryptage symétrique AES-256
Symétrique signifie que la clé de cryptage est la même que celle de décryptage : la longueur des clés est de 256 bits (32 octets).

Personne en 2023 n'a proclamé avoir réussi à casser un cryptage de clé aléatoire AES-256.

Le cryptage / décryptage est rapide et peut concerner des textes de n'importe quelle taille.

### Cryptage asymétrique RSA-2048
Un couple de clés est généré simultanément :
- la **clé publique** est utilisée pour crypter un texte de longueur maximale de 256 octets. Même quand le texte origine est plus court, le texte crypté occupe 256 octets.
- la **clé privée** est utilisée pour décrypter un texte crypté par la clé publique.

Le cryptage / décryptage est lent. On utilise ce cryptage pour produire un texte lisible seulement par le détenteur de la clé privée. De fait la clé publique est comme son nom l'indique disponible à n'importe qui.

Comme la longueur du texte à crypter est courte, on utilise souvent le cryptage asymétrique pour crypter ... une clé de cryptage symétrique.

### Brouillage PBKFD2
Cet algorithme _brouille_ un texte initial pour en restituer un texte court (32 octets) : il est quasi impossible de retrouver le texte initial depuis le texte brouillé en raison du coût élevé de calcul que ça demande et de l'impossibilité d'utiliser des processeurs spécifiques ou graphiques à cet effet.

> L'inconvénient de cet algorithme est le pendant de son avantage : 2 secondes d'attente pour un calcul, ça demande une conception qui en tienne compte et exclut pratiquement un usage sur un serveur.

PBKFD2 est employé pour brouiller les phrases secrètes qui, vu leur longueur, ne peuvent pas être pas être cassées par _force brute_ avant la fin de la planète. L'usage de dictionnaires de mots de passe fréquent a montré son inefficacité dès lors que le texte est long (24 signes c'est beaucoup) et qu'on y alterne majuscules, minuscules, chiffres séparateurs, voir quelques caractères spéciaux.

### Fonction hash
Cette fonction prend en entrée une suite d'octets et en retourne un entier de 53 bits avec peu de _collisions_ : deux textes différents ne donnent le même hash, en gros qu'une fois sur un million de milliards de fois.

## A propos des comptes et leurs avatars

### Clé et identifiant d'un avatar
La **clé** d'un avatar est constituée de 32 octets (256 bits) tirés au sort à la création de l'avatar. Elle crypte sa _carte de visite_.

Son identifiant est basé sur le _hash_ de sa clé.

Par exception, l'identifiant du Comptable est `1000000000000000` pour l'espace `10`.

### Identifiant d'un compte
Un compte et son avatar principal ont le même identifiant.

### Clé K du compte
Cette clé de cryptage AES-256 (32 octets) a été tirée aléatoirement par l'application à la création du compte :
- c'est la clé majeure du compte : elle est immuable et crypte toutes les données cruciales du compte.
- elle ne transite jamais en clair sur le réseau, n'est jamais communiqué au serveur et reste dans la mémoire de l'application durant son exécution.
- elle est conservée dans l'enregistrement du compte cryptée par un brouillage PBKFD2 de la phrase secrète du compte. Le changement de phrase secrète se limite de ce fait à ré-encrypter la clé K.

### Informations détenues sur le compte et ses avatars
Dans son enregistrement maître :
- la **clé K** cryptée par le brouillage de la phrase secrète.
- le **nom et la clé de la tribu** du compte, cryptés par la clé K.
- la **liste des avatars principal et secondaires** du compte, cryptée par la clé K. Pour chaque avatar, le compte détient :
  - son _nom_, sa _clé_ et son _identifiant_.
  - la _clé privée RSA_ attribuée à l'avatar a sa création.
- la **liste des mots-clés** déclarés par le titulaire du compte, cryptée par la clé K. En conséquence seul le titulaire du compte les voit, ils ne sont jamais publics et peuvent ainsi déroger à toutes les règles de civilité en usage.
- le **mémo** privé du compte, crypté aussi par la clé K.
- pour chaque avatar, 
  - **la liste des groupes dont l'avatar est membre** est également cryptée par la clé K. La clé de cryptage d'un groupe est ainsi elle-même enchâssée dans un item crypté.
  - sa **carte de visite**, petite photo et court texte, cryptés par la clé de l'avatar.