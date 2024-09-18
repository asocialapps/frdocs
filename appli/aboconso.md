---
layout: page
title: Abonnement, consommation, tarifs
---

Le coût d'usage de l'application pour une organisation correspond aux coûts d'hébergement des données et de traitement de celles-ci. Selon les techniques et les prestataires choisis, les coûts unitaires varient mais existent dans tous les cas.

## _Base de données_ et _fichiers_ (Storage)
Leur stockage ont des coûts unitaires très différents (variant d'un facteur de 1 à 6).
- les _bases de données_ requièrent un stockage proche du serveur et des accès très rapides,
- les fichiers sont enregistrés dans des _Storage_, des stockages techniques distants ayant une gestion spécifique et économique du fait d'être soumis à peu d'accès (mais de plus fort volume).

## Abonnement: coût de l'espace occupé en permanence
Un abonnement correspond aux coûts récurrents mensuels pour un compte:  même quand il ne se connecte pas, le stockage de ses sonnées a un coût.

L'abonnement est décomposé en deux lignes de coûts correspondant à l'occupation d'espace en base de données et en _storage_:
- **Prix unitaire de stockage d'un document** multiplié par le **nombre total de _documents_**: notes personnelles et notes d'un groupe hébergé par le compte, chats personnels non _indésirables_, nombre de participations actives aux groupes.
- **Prix unitaire du stockage des fichiers dans un _storage_** multiplié par le **volume total des fichiers attachés aux notes**.

Pour obtenir le coût correspondant à ces deux volumes il est pris en compte, non pas _le volume effectivement utilisé à chaque instant_ mais forfaitairement **les _volumes maximaux_ forfaitaires** auquel le compte est abonné (ses _quotas_).

> Les volumes _effectivement utilisés_ ne peuvent pas dépasser les volumes maximum de l'abonnement. Dans le cas où un changement de l'abonnement réduit a posteriori ces maximum en dessous des volumes utilisés, les volumes n'auront plus le droit de croître.

## Consommation : coût de calcul et de transfert des fichiers
La consommation correspond à l'usage effectif de l'application quand une session d'un compte est ouverte. Elle comporte 4 lignes:
- **nombre _de lectures_** (en base de données).
- **nombre _d'écritures_** (en base de données).
- **volume _descendant_** (download) de fichiers téléchargés en session depuis le _storage_.
- **volume _montant_** (upload) de fichiers envoyés dans le _storage_ pour chaque création / mise à jour d'un fichier.

## Coût total mensuel
Il correspond au total de l'abonnement (2 lignes) et de la consommation (4 lignes) valorisées par un **tarif** fixé par le prestataire:
- un tarif est simplement la données des 6 coûts unitaires des 6 éléments d'abonnement (2) et de consommation (4).

>_L'ordre de grandeur_ du prix de revient total pour un compte varie en gros de **0,5€ à 3€ par an**. Individuellement ça paraît faible mais n'est plus du tout négligeable pour une organisation assurant les frais d'hébergement d'un millier de comptes ...

# Qui paye quoi ?
**Les comptes _autonomes_ "A" payent leur abonnement et leur consommation:**
- ils ont à tout instant un _solde_,
  - somme de tous les _paiements_ qui ont été enregistrés,
  - diminuée de l'encours de leur abonnement à tout instant et de leur consommation à chaque opération.

Les comptes "A" peuvent _s'auto-limiter_ en se fixant des limites à ne pas dépasser sur leur _abonnement_ : nombre de _documents_ maximum, volume de fichiers maximum. Ils sont facturés selon ces _maximum_ (quotas) et non leur usage effectif.

**Les comptes _de l'organisation_ "A" ne payent, ni leur abonnement, ni leur consommation:**
- l'organisation (le Comptable et ses _délégués_) fixent des quotas, des maximum d'abonnement (nombre de documents, volume de fichiers) et un maximum de consommation mensuelle.
- les dépassements de volumes sont bloqués.
- l'approche du maximum de consommation provoque des **ralentissements**.

L'organisation payent donc de facto pour ses comptes "O". Le Comptable est chargé de retourner au prestataire les montants encaissés des comptes "A", 
- soit globalement et périodiquement selon un processus à établir entre eux,
- soit en encaissant les paiements directement sur un compte du prestataire.

# Gestion de l'espace (_abonnements gratuits_) des comptes "O"
Le Comptable procède d'abord à un _découpage en tranches_ des ressources globales dont il dispose:
- chaque _tranche_ a un quota de _nombre de documents_, de _volume de fichiers_ et de _consommation de calcul_.
- tout compte "O" est attaché à une _tranche_.
- pour chaque _tranche_ le Comptable peut (ou non) confier une _délégation_ à certains comptes de la tranche afin que ceux-ci,
  - fixent pour chaque compte "O" de leur tranche des quotas d'abonnement et de consommation,
  - puissent gérer des _notifications_ à ces comptes (avec restriction éventuelle).
