---
layout: page
title: Abonnement, consommation, tarifs
---

Le coût d'usage de l'application pour une organisation correspond aux coûts d'hébergement des données et de traitement de celles-ci. Selon les techniques et les prestataires choisis, les coûts unitaires varient mais existent dans tous les cas.

## _Base de données_ et _fichiers_ (Storage)
Leur stockage ont des coûts unitaires très différents (variant d'un facteur de 1 à 8).
- les _bases de données_ requièrent un stockage proche des serveurs de traitement et des accès rapides,
- les fichiers sont enregistrés dans des _Storage_, des stockages techniques distants ayant une gestion spécifique et économique du fait d'être soumis à peu d'accès (mais de plus fort volume).

## Abonnement: coût de l'espace occupé en permanence
Un abonnement correspond aux coûts récurrents mensuels pour un compte:  même quand il ne se connecte pas, le stockage de ses sonnées a un coût.

L'abonnement couvre les frais fixes d'un compte: même quand il ne se connecte pas, le stockage de ses données a un coût. Il est décomposé en deux lignes de coûts correspondant à l'occupation d'espace forfaitaire en _base de données_ et en _storage_:
- _Coût 1_ : **prix unitaire de stockage d'un document** multiplié par le **le nombre maximal du quota de _documents_ de l'abonnement**: notes personnelles et notes d'un groupe hébergé par le compte, chats personnels, nombre de participations actives aux groupes.
- _Coût 2_ : **prix unitaire du stockage des fichiers dans un _storage_** multiplié par le **le volume maximal du quota de l'abonnement des fichiers attachés aux notes**.

> Pour obtenir le coût correspondant à ces deux volumes il est pris en compte, non pas _le volume effectivement utilisé à chaque instant_ mais forfaitairement **les _quotas_ (_volumes maximaux_)** auquel le compte est abonné.

> Les volumes _effectivement utilisés_ ne peuvent pas dépasser les quotas (volumes maximum) déclarés pour l'abonnement. Dans le cas où un changement de l'abonnement réduit a posteriori ces maximum en dessous des volumes utilisés, les volumes n'auront plus le droit de croître.

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

## Annexe: estimation d'un _tarif_ depuis les prix de marché
Il n'est considéré ci-après que l'estimation d'un _tarif_ basé sur la seule consommation de services externes de Cloud: chaque prestataire doit ensuite y ajouter le prix de sa valeur ajoutée, commercialisation, support technique, etc.

**Les unités d'œuvre d'abonnement et de consommation ont été simplifiées pour être compréhensible par les comptes**. Par exemple:
- les coûts de _calcul_ (appel d'une fonction Cloud, heure de serveur ...) sont facturés par le fournisseur Cloud mais sont intégrés dans les coûts des lectures et écritures.
- les coûts de _download_ comporte une partie fixe par opération et une partie de coût réseau dépendante du volume: la partie fixe est à intégrer dans la part variable liée au volume.

### Prix relevés chez Google (Firestore / Storage)
Ce sont des ordres de grandeur, les vrais coûts sont complexes, dépendent de la localisation des serveurs, etc.

Prix mensuels du stockage (abonnement)
- Base de Données (1GB) : 16c
- Storage (1GB) : 2.6c

Prix des consommations:
- Lectures en base (1M): 31c
- Écritures en base (1M): 94c
- Download (1GB): 2c

### Exemple de _tarif_ basé sur ces prix
Les quotas sont exprimés en _nombre d'unités_ de 0 à 200 (exceptionnellement plus), afin de manipuler des ordres de grandeurs simples: 
- 1 est un minimum réaliste, 
- 200 est un quota très important.

#### Prix d'abonnement mensuel par _document_
Un document est une note, un chat, une participation à un groupe. Le volume moyen est estimé à 10K par document, englobant les index et les autres documents non décomptés (les comptes, avatars, groupes, etc.).

**Une _unité_ de 100 documents** correspond à un volume en base de 2.5MB: **0,02c mensuels**

#### Prix d'abonnement mensuel par _volume de fichier_
**Une _unité_ de 100MB de _Storage_** : **0,26c mensuels**

#### Consommations de calcul
Les prix brut du tarif Google ont été multipliés par 2 pour tenir comptes des parts fixes par opération, des coûts réseaux non décomptés, des coûts des appels de _Cloud function / Heure de serveur_.

Les 4 unités retenues sont, 
- le nombre de millions de lectures: 60c
- le nombre de millions d'écritures: 200c
- le volume en GB de fichiers _descendants_: 4c
- le volume en GB de fichiers _montants_: 4c

**Le quota de _consommation mensuelle_ d'un compte "0"** est donné en `c` :
- un million de lectures ou d'écritures par mois correspond à une très très grosse activité.
- en revanche télécharger quelques GB de fichiers est _plausible_.
- le quota _minimal_ de 1c correspond, a priori, à une activité faible.
- un quota de 200c par mois correspond, a priori, à une utilisation intense, surtout en téléchargements.
