---
layout: page
title: Maîtrise des coûts d'hébergement de l'application
---

L'unité monétaire interne est le _centime_ (c). Son ordre de grandeur est voisin d'un centime d'euro en appliquant des tarifs proches des coûts _de base_ des hébergements sur le marché. Chaque fournisseur a bien entendu ses propres tarifs qui peuvent être significativement éloignés des coûts _de base_.

Le coût d'usage de l'application pour une organisation correspond aux coûts d'hébergement des données et de traitement de celles-ci. Selon les techniques et les prestataires choisis, les coûts unitaires varient mais existent dans tous les cas.

### _Base de données_ et _fichiers_ (Storage)
Leur stockage ont des coûts unitaires très différents (variant d'un facteur de 1 à 6).
- les _bases de données_ requièrent un stockage proche du serveur et des accès très rapides,
- les fichiers sont enregistrés dans des _Storage_, des stockages techniques distants ayant une gestion spécifique et économique du fait d'être soumis à peu d'accès (mais de plus fort volume).

### Abonnement: coût de l'espace occupé en permanence
L'abonnement couvre les frais fixes d'un compte:  même quand il ne se connecte pas, le stockage de ses données a un coût. Il est décomposé en deux lignes de coûts correspondant à l'occupation d'espace en _base de données_ et en _storage_:
- **Prix unitaire de stockage d'un document** multiplié par le **nombre de _documents_**: notes personnelles et notes d'un groupe hébergé par le compte, chats personnels non _indésirables_, nombre de participations actives aux groupes. Une note ayant N fichiers de type _image_ attachés est décompté pour N+1 afin de tenir compte du volume en base de l'image miniature.
- **Prix unitaire du stockage des fichiers dans un _storage_** multiplié par le **volume des fichiers attachés aux notes**.

Pour obtenir le coût correspondant à ces deux volumes il est pris en compte, non pas _le volume effectivement utilisé à chaque instant_ mais forfaitairement **les _quotas_ (_volumes maximaux_)** auquel le compte est abonné.

> Les volumes _effectivement utilisés_ ne peuvent pas dépasser les quotas (volumes maximum) déclaré pour l'abonnement. Dans le cas où un changement de l'abonnement réduit a posteriori ces maximum en dessous des volumes utilisés, les volumes n'auront plus le droit de croître.

### Consommation : coût de calcul et de transfert des fichiers
Le coût de calcul apparaît lors de l'usage effectif de l'application quand une session d'un compte est ouverte. Elle comporte 4 lignes:
- **nombre _de lectures_** (en base de données).
- **nombre _d'écritures_** (en base de données).
- **volume _descendant_** (download) de fichiers téléchargés en session depuis le _storage_.
- **volume _montant_** (upload) de fichiers envoyés dans le _storage_ pour chaque création / mise à jour d'un fichier.

### Coûts réel et facturé
Le coût _réel_ d'un compte sur une période donnée correspond à la somme du coût _d'abonnement_ et du coût de _consommation_ pendant cette période.

Le coût _facturé_ sur une période est nul pour un compte "O" et est le coût _réel_ pour un compte "A".

Le solde d'un compte est calculé à tout instant,
- en le créditant des paiements et dons reçus dans la période,
- en le débitant des dons effectués et des coûts _facturés_ dans la période.

> Les tarifs peuvent changer d'un mois à l'autre, mais sont fixes dans un mois donné.

Un compte peut afficher à tout instant,
- l'état de sa comptabilité **calculée à l'instant de l'affichage**: quotas, nombre de documents et volumes de fichiers effectivement occupés, **solde courant**.
- l'historique de l'évolution mois par mois sur les 12 derniers mois: moyennes des quotas, des consommations, coût total sur le mois (réel et facturé), débits et crédits, soldes en début et fin de mois.

## Prix unitaires de marché
_L'ordre de grandeur_ des prix du marché, donne les coûts suivants en centimes d'euro annuel:

    1 Gb Firestore      216 c / an
    1 Gb GCP Storage     32 c / an (6 fois moins)

    Pour une "unité" de quota : 2Mo (100 notes chats groupes)  -> 0,21c/an
    Pour une "unité" de quota : 100Mo                          -> 3,20c/an

> Le volume des _documents_ est difficile à estimer puisqu'il s'agit du volume _technique_ avec les index, clés, etc. Forfaitairement il a été pris 20K par document. Le volume des _fichiers_ est assez proche du volume utile.

> Les volumes des _documents_ apparaissent environ 6 fois plus coûteux au méga-octet que les volumes _fichiers_, mais comme les fichiers peuvent être très volumineux, le coût d'utilisation dépend de ce que chacun met en textes des notes et en fichiers attachés.

## La consommation de calcul dépend de la façon dont chacun se sert de l'application

Ces coûts de _calcul_ correspondent directement à l'usage fait de l'application quand une session d'un compte est ouverte. Ils dépendent de _l'activité_ du titulaire du compte et de la façon dont il se sert de l'application. Le coût de calcul est la somme de 4 facteurs, chacun ayant son propre tarif:
- **(nl) nombre de _lectures_** (en base de données): nombre de notes lues, de chats lus, de contacts lus, de membres de groupes lus, etc. **Lu** signifie extrait de la base de données.
  - en utilisant un mode _synchronisé_ la grande majorité des données étant déjà présentes sur l'appareil du titulaire du compte, les _lectures_ se résument à celles des données ayant changé ou ayant été créés depuis la fin de la session précédente. 
  - pour un même service apparent, le coût des _lectures_ peut varier, par exemple par rafraîchissement systématique des cartes de visite, ou en gardant un chat en ligne au lieu de raccrocher ...
  - en mode _avion_ le nombre de lectures est par principe nul.
- **(ne) nombre _d'écritures_** (en base de données): outre quelques écritures de gestion du compte, elles correspondent principalement aux _mises à jour des notes, chats, cartes de visite, commentaires personnels, gestion d'un groupe_.
- **(vd) volume _descendant_** (download) de fichiers téléchargés depuis le _Storage_. Chaque acte est explicitement demandé par le titulaire du compte: 
  - quand il utilise une _copie locale_ d'un fichier sur son appareil en mode _synchronisé_, le volume téléchargé est nul, sauf si le fichier a changé. 
  - il est possible de télécharger sur son appareil toutes les notes d'une sélection faite à l'écran (et potentiellement toutes celles accessibles par le compte): en mode _synchronisé_ ça ne coûte rien, SAUF s'il a été demandé de télécharger aussi les fichiers leur étant attachés ce qui occasionne un coût de volume descendant qui peut être important.
- **(vm) volume _montant_** (upload) de fichiers envoyés dans le _Storage_. Chaque création / mise à jour d'un fichier est décompté dans ce volume montant.

_L'ordre de grandeur_ des prix de marché donne les coûts unitaires suivants en euros:

    100000 lectures   ->  8c 
    100000 écritures  -> 20c 
    Transfert d'un GB -> 15c

# Les tarifs
Les tarifs sont donnés en configuration du serveur et comporte une ligne par mois auquel le tarif a changé (un tarif s'applique par mois entier).

Les coûts unitaires en centimes sont donnés pour les 6 compteurs:
- 100 documents
- 1Gb de fichiers
- 100000 lectures
- 100000 écritures
- 1Gb de transfert descendant (download)
- 1Gb de transfert montant (upload)

      tarifs: [
        { am: 202401, cu: [0.45, 0.10, 8, 20, 15, 15] },
        { am: 202501, cu: [0.55, 0.15, 8, 18, 15, 15] },
        { am: 202506, cu: [0.65, 0.10, 8, 15, 15, 15] }
      ]

# Procédé de calcul de la comptabilité à l'instant t
La structure est la suivante:
- `dh0` : date-heure de création du compte
- `dh` : date-heure de calcul actuelle
- `dhP` : date-heure de création ou de changement O <-> A (informative, n'intervient pas dans les calculs).
- `estA` : `true` si le compte est A.
- `qv` : quotas et volumes du dernier calcul `{ qc, qn, qv, nn, nc, ng, v, cjm }`.
- `ddsn` : date-heure de début de solde négatif.
- `vd` : [0..11] - vecteur détaillé pour les 12 mois de l'année (glissante)

Propriétés calculées:
- `cjm` : consommation moyenne de M M-1 ramenée à un jour.
- `njec` : nombre de jours estimés avant épuisement du crédit.
- `flags` : flags courants 
  - `RAL` : ralentissement (excès de calcul / quota)
  - `NRED` : documents en réduction (excès de nombre de documents / quota)
  - `VRED` : volume de fichiers en réduction (excès de volume / quota)
  - `ARSN` : accès restreint pour solde négatif
- `aaaa mm` : année / mois de dh.

**Vecteur `vd`** : pour chaque mois M de l'année glissante ([0] est janvier)
- MS 0 : nombre de ms dans le mois - si 0, le compte n'était pas créé
- moyennes des quotas:
  - QC 1 : moyenne de qc dans le mois (en c)
  - QN 2 : moyenne de qn dans le mois (en nombre de documents)
  - QV 3 : moyenne de qv dans le mois (en bytes)
- cumuls des consommations:
  - NL 4 : nb lectures cumulés sur le mois (L),
  - NE 5 : nb écritures cumulés sur le mois (E),
  - VM 6 : total des transferts montants (B),
  - VD 7 : total des transferts descendants (B).
- moyennes des compteurs:
  - NN 8 : nombre moyen de notes existantes.
  - NC 9 : nombre moyen de chats existants.
  - NG 10 : nombre moyen de participations aux groupes existantes.
  - V 11 : volume moyen effectif total des fichiers stockés.
- compteurs monétaires
  - AC 12 : coût de l'abonnement (dans le mois)
  - AF 13 : abonnement facturé (dans le mois)
  - CC 14 : coût de consommation (dans le mois)
  - CF 15 : consommation facturée (dans le mois)
  - DB 16 : débits du mois
  - CR 17 : crédits du mois
  - S 18 : solde au début du mois
  
Le solde en fin de mois est celui du début du mois suivant: S - DB + CR - CF - AF

Le principe de calcul est de partir avec la dernière photographie enregistrée à la date-heure `dh`.
- le calcul démarre _maintenant_ à la date-heure `now`.
- la première étape est d'établir le passé survenu entre `dh` et `now`: ce peut être quelques secondes comme 18 mois.
  - par principe aucun événement ne s'est produit entre ces deux instants, il s'agit donc de _prolonger_ l'état connu à `dh` jusqu'à `now`.
  - le mois M de la photo précédente à `dh` doit être prolongé, soit jusqu'à `now`, soit jusqu'à la fin du mois.
  - puis le cas échéant il _peut_ y avoir N mois entiers à prolonger dans l'état connu à fin M.
  - puis le cas échéant il _peut_ y avoir un dernier mois incomplet prolongeant le dernier calculé.

Quand on prolonge un mois selon les compteurs deux cas se présentent:
- soit c'est une addition : les nombres de lectures, écritures ... augmentent.
- soit c'est l'ajustement d'une _moyenne_ en fonction du nombre de millisecondes sur laquelle elle était calculée et celui sur laquelle elle est à prolonger.

Le calcul s'effectuant depuis le dernier mois calculé, mois par mois, le calcul peut s'effectuer sur plus de 12 mois, sachant que les onze derniers et le mois courant sont disponibles dans `vd`.

Après la phase de prolongation de dh à now, on met à jour le nouvel état courant:
- les compteurs `qv` peuvent être à mettre à jour,
- le statut O/A peut être à mettre à jour,
- une consommation peut être à enregistrer: c'est au cycle suivant qu'elle _coûtera_.

Le coût de calcul moyen sur M M-1 peut être effectué: si le nombre de ms de cette période est trop faible (moins de 10 jours) la moyenne peut être aberrante en survalorisant les opérations les plus récentes. Cette moyenne considère qu'il y a toujours eu au moins 10 jours de vie, même si la création remonte à moins que cela.
