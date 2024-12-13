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

    'IP',
    'NJ', 
    'QC', 'QN', 'QV', 
    'NL', 'NE', 'VM', 'VD', 
    'NN', 'NC', 'NG', 'V', 
    'AC', 'AF', 'CC', 'CF'

    - IP : ID de la partition du compte ou '' pour un compte "A".
    - NJ 0 : nombre de jours d'existence du compte dans le mois.
    moyennes des quotas:
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

> La somme des AF+CF indique ce que les comptes devraient avoir payé (tant qu'ils étaient compte A dans ce mois).

> La somme des AC+CC indique ce que les comptes _ont coûté_ à l'organisation.

> Le rapprochement de `NN+NC+NG` avec `QN` permet de comparer l'usage effectif par rapport aux quotas (de même entre `V` et `QV`).

## Archive des _tickets_ de paiement
Cette archive (un fichier CSV) contient une ligne par _ticket de paiement_ émis / reçu dans le mois. Le ticket est anonyme, il n'est pas possible d'effectuer un rapprochement avec le compte qui l'a émis.

Elle permet au Comptable de totaliser ce qu'il a reçu en paiement, du moins ce qu'il a enregistré avoir reçu.

Colonnes:

    IDS,TKT,DG,DR,MA,MC,REFA,REFC

- `IDS` : identifiant du ticket commençant par `aamm` (l'année et le mois)
- `TKT` : numéro du ticket.
- `DG` : date de génération sous la forme `aaaammjj`
- `DR`: date de réception / enregistrement sous la forme `aaaammjj`. Si 0 le ticket est _en attente_.
- `MA`: montant en `c` déclaré émis par le compte A.
- `MC` : montant en `c` déclaré reçu par le Comptable.
- `REFA` : référence facultative écrite par le compte A à l'émission.
- `REFC` : référence facultative écrite par le Comptable à la réception.

