---
layout: page
title: Statistiques partagées entre le Comptable et l'administrateur technique
---

Tous deux ont besoin d'éléments statistiques de consommation afin d'ajuster si nécessaire les bases de facturation en fonction d'un usage réel.

Ces statistiques sont calculées mensuellement et sont des fichiers CSV décrits ci-après.
- elles sont anonymes: les identifiants des comptes concernés n'y figurent pas.
- pour un mois donné elles sont immuables, calculées à un moment où le mois étant terminé, les compteurs mensuels ne sont plus susceptibles de changer.

## Abonnement / consommation des comptes
Une ligne (anonyme) par compte comportant les colonnes suivantes:

    qc, qn, qv, nl, ne, vm, vd, nn, nc, ng, v, nj, ca, cc

Abonnement:
- `qc` : moyenne de la consommation mensuelle de calcul dans le mois en `c`. Une valeur à 0 correspond à un compte "A" qui n'a pas par principe d'abonnement de _calcul_.
- `qn` : moyenne de l'abonnement en nombre de documents (en unité de 250 documents).
- `qv` : moyenne de l'abonnement en volume de fichiers (en unité de 100Mo).

Consommation:
- `nl` : nombre de lectures cumulés sur le mois.
- `ne` : nombre d'écritures cumulés sur le mois.
- `vm` : volume total des transferts de fichiers montants.
- `vd` : volume total des transferts de fichiers descendants.

Moyennes d'usage:
- `nn` : nombre moyen de notes.
- `nc` : nombre moyen de chats.
- `ng` : nombre moyen de participations aux groupes.
- `v` : volume moyen effectif total des fichiers stockés.

Compteurs spéciaux:
- `nj` : nombre de jours d'existence du compte dans le mois.
- `ca` : coût de l'abonnement pour le mois.
- `cc` : coût de la consommation pour le mois.

> La somme des ca et cc indique ce que les comptes ont payé (pour les comptes "A") ou _coûté_ à l'organisation (pour les comptes "O").

> Le rapprochement de `nn + nc + ng` avec `qn` permet de comparer l'usage effectif par rapport à l'abonnement. De même entre `qv` et `v`.

## Archive des _tickets_ de paiement des comptes "A"
Cette archive contient une ligne par _ticket de paiement_ émis / reçu dans le mois. Le ticket est anonyme, il n'est pas possible d'effectuer un rapprochement avec le compte qui l'a émis.

Elle permet au Comptable de totaliser ce qu'il a reçu en paiement, du moins ce qu'il a enregistré avoir reçu.

Colonnes:

    ids, dg, dr, ma, mc, refa, refc

- `ids` : identifiant du ticket commençant par `aamm` (l'année et le mois)
- `dg` : date de génération sous la forme `aaaammjj`
- `dr`: date de réception / enregistrement sous la forme `aaaammjj`. Si 0 le ticket est _en attente_.
- `ma`: montant en `c` déclaré émis par le compte A.
- `mc` : montant en `c` déclaré reçu par le Comptable.
- `refa` : référence facultative écrite par le compte A à l'émission.
- `refc` : référence facultative écrite par le Comptable à la réception.

