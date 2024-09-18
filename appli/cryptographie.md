---
layout: page
title: Quelques principes de cryptographie
---

# L'usage de la cryptographie dans a-social

Plusieurs principes ont été mis en œuvre:
- tous les textes humainement intelligibles et les images sont cryptées.
- toutes les clés de cryptage pour chaque compte sont cryptées par la clé majeure K du compte, elle-même cryptée par la phrase secrète du compte.
- aucune clé de cryptage n'est transmise au serveur.

# Technologies de cryptage / hash employées
## Hash: SHA256
Ce hash d'une suite de bytes à une longueur de 32 bytes.

Il n'a pas été publié de cas où deux entrées différentes produisaient le même SHA256 (ce qui mathématiquement est possible).

SHA256 a un avantage qui est aussi un inconvénient: il est rapide à calculer et l'usage de processeurs graphiques accélèrent son calcul.
- tout est relatif: une attaque par force brute (essai de toutes les combinaisons possibles) pour tenter de retrouver le texte source depuis son SHA256 devient impraticable pour des sources d'une longueur De 20 bytes et au-delà, l'énergie consommée en calcul devenant inaccessible.
- il n'en reste pas mois que SHA256 n'est pas approprié pour hacher des mots de passe / phrase secrète.

## Hash : PBKFD
Ce hash d'une suite de bytes à une longueur de 32 bytes.

Il n'a pas été publié de cas où deux entrées différentes produisaient le même PBKFD (ce qui mathématiquement est possible).

Le PBKFD est _long_, volontairement, et son algorithme n'est pas accéléré par l'usage de processeurs graphiques.

PBKFD est utilisé pour hacher les phrases secrètes et de rencontre:
- avec une source imposée à 24 signes au minimum, l'attaque par force brute est impossible.
- mais casser un mot de passe se pratique aussi en utilisant des _dictionnaires_ de mots usuels. 

Une phrase secrète de 24 signes qui évite les répétitions d'un code court, qui parsème le texte de séparateurs en chiffres, etc. ne sera pas trouvée par usage de dictionnaires.

## Hash court
C'est seulement un repliement d'un SHA256 sur 9 bytes, soit 12 caractères chiffres ou lettres miniscules / majuscules.

## Cryptage symétrique AES256
La même clé de 32 bytes est employée pour crypter et décrypter un texte.
- une autre clé dite _SALT_ est employée pour compliquer le piratage: il faut avoir la clé et le _SALT_ pour décrypter un texte.
- les _SALT_ sont usuellement dans le code.
- le cryptage est rapide et les texte crypté peut avoir n'importe quel longueur.

Il n'a pas été rendu public qu'un texte source (long) ait pu être retrouvé par application de force brute sur sa clé de cryptage.

Le premier byte d'une chaîne cryptée est le numéro du _SALT_ employé au cryptage:
- il suffit donc que les logiciels cryptant / décryptant utilisent la même liste de _SALT_.
- en tirant au hasard ce premier byte, une même source cryptée deux fois aura des cryptages différents 255 fois sur 256: ainsi la comparaison entre deux chaînes cryptées n'indiquent pas si la source est la même ou non.
- toutefois quand une chaîne cryptée sert d'identifiant externe, il faut à l'inverse que la même source donne toujours le même résultat crypté: ceci s'obtient en forçant le cryptage à utiliser le _SALT_ premier de la liste plutôt qu'un aléatoire.

## Cryptage asymétrique RSA2048
Le cryptage nécessite une paire de clés générées ensemble:
- la clé publique sert à crypter un texte qui ne pourra être décrypté qu'en utilisant la clé privéE. Les deux clé sont longues.
- le texte _source_ a une longueur maximale de 256 bytes.
- le cryptage est lent.
- deux cryptages successifs d'un même texte source donne deux cryptages différents.

> 2048, c'est 256 bytes. Attaquer par force brute une telle clé semble irréaliste.

L'objectif est que le _public_ puisse librement crypter un texte à destination d'un seul destinataire.

Quand le texte à crypter peut être plus long que 256 bytes, on génère une clé symétrique aléatoire qui crypte le long texte et on crypte cette clé par la clé RSA.

Chaque avatar a un couple de clés RSA, tout un chacun ayant accès à la la clé publique peut crypter un texte que seul l'avatar cible peut décoder.

# Les clés d'un compte

## Phrase secrète
Quand un compte définit sa phrase secrète on s'assure que celle-ci n'a pas déjà été employée en utilisant son PBKFD. 

MAIS, si on donne à quelqu'un l'information que la phrase a déjà été employée, c'est lui donner l'assurance qu'il peut accéder à un autre compte.

Pour chaque phrase on calcule deux PBKFD,
- celui de phrase complète,
- celui d'une phrase réduite en ne prenant que certains signes de la phrase complète.
- on exclut la possibilité de choisir une phrase secrète dont le PBKFD de la phrase raccourcie a déjà été employée.

Certes le compte sait ainsi qu'un autres compte a une phrase secrète dont certains signes à des positions données (lisibles dans le code) lui sont connus. Mais ses tentatives pour trouver les autres sont d'un coût rédhibitoire, d'autant plus qu'il n'en connaît pas la longueur.

## Clé "K" d'un compte
C'est une clé de 32 bytes tirée au hasard: il est impensable d'essayer de la retrouver par force brute.

Elle est stockée dans le document majeur du compte cryptée par le PBKFD de la phrase secrète:
- toutes les autres clés générées sont stockées cryptées par cette clé.
- on peut changer de _phrase secrète_ facilement: le changement re-crypte la clé K mais ne la change pas.

> La clé K n'est jamais communiquée en dehors d'une session. Elle figure en clair dans la session du compte puisqu'elle sert en permanence à obtenir d'autres clés. Mais le compte connaît SA _phrase secrète_ ... fouiller la mémoire pour trouver la clé K n'a aucun intérêt.

La base locale d'un compte sur un poste est une base de type _clé / valeur_.
Les clés comme les valeurs sont cryptées par la clé K:
- celle-ci est rangée dans cette base cryptée par le PBKFD de la phrase secrète.
- pirater une base locale ne sert à rien: soit on n'a pas la clé et on ne la trouvera pas, soit on l'a ... et on a pas besoin de pirater des données auxquelles on peut accéder en clair par l'application.

La clé K crypte toutes les notes des avatars du compte.

## Clé "CV" d'un avatar
Quand un avatar est créé, il est générée une clé dite CV qui crypte sa carte de visite. La clé CV est mémorisée dans le document maître de l'avatar cryptée par la clé K du compte.

Cette clé est aussi incassable que les autres sauf qu'elle va être donnée à tous les autres avatars _en contact_, ceux qui justement on a accordé le droit de lire son nom et sa carte de visite.

> La **fragilité** n'est pas dans le procédé cryptographique mais dans le nombre de _contacts_ à qui on a attribué sa confiance.

Cela dit le risque se limite à ce que des contacts pas forcément désirables un jour puissent lire votre nom et votre carte de visite. Les autres données sont définitivement inaccessibles aux autres avatars.

## Clé d'un chat entre deux avatars
Elle est générée aléatoirement à la création du chat et est cryptée par les clés K respectives des deux comptes. A la création, le créateur du chat crypte cette clé par la clé publique RSA de l'autre: il n'y a que lui qui puisse la décoder et ré-encrypter la clé du chat par sa propre clé K (ce qui sera fait à sa prochaine ouverture de session).

## Clé d'un groupe
Elle est générée aléatoirement à la création du groupe.
- chaque membre du groupe la reçoit lors de son invitation cryptée par sa clé RSA publique.
- il ré-encrypte cette clé par sa clé K plus tard dans une session ouverte.

La clé d'un groupe crypte aussi son nom et sa carte de visite.

La clé du groupe crypte toutes les notes du groupe.

> La **fragilité** n'est pas dans le procédé cryptographique mais dans le nombre de _membres_ à qui le groupe a attribué sa confiance.

## Clé d'un sponsoring
Un sponsoring est créé par un avatar pour être lu par ... une personne qui n'a pas de compte. Il est donc impossible d'utiliser une clé RSA qu'il na pas encore.

On utilise le PBKFD de la phrase de sponsoring comme clé de cryptage:
- seul le destinataire la connaît (du moins c'est plus poli),
- le document de sponsoring ne vit pas longtemps et il est inutilisable une fois qu'il a servi à créer un compte sponsorisé.

## Données NON cryptées
Certaines données ne sont pas cryptées tout le temps. Par exemple un Ticket de crédit:
- son identifiant est généré aléatoirement.
- pour le Comptable il accède à ces tickets _en clair_ mais est incapable de savoir qui a généré ce ticket.
- pour le compte émetteur les tickets sont stockés cryptés par sa clé K.

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