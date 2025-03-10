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
    - NJ : nombre de jours d'existence du compte dans le mois.

    Moyennes des quotas:
    - QC : moyenne de qc dans le mois (en c)
    - QN : moyenne de qn dans le mois (en nombre de 100 documents)
    - QV : moyenne de qv dans le mois (en 100 Mb)

    Cumuls des consommations:
    - NL : nb lectures cumulés sur le mois (L),
    - NE : nb écritures cumulés sur le mois (E),
    - VM : total des transferts montants (B),
    - VD : total des transferts descendants (B).

    Moyennes des compteurs:
    - NN : nombre moyen de notes existantes.
    - NC : nombre moyen de chats existants.
    - NG : nombre moyen de participations aux groupes existantes.
    - V  : volume moyen effectif total des fichiers stockés.
    
    Compteurs monétaires en CENTIÈMES de c (150 => 1,5c)
    - AC : coût de l'abonnement (dans le mois)
    - AF : abonnement facturé (dans le mois)
    - CC : coût de consommation (dans le mois)
    - CF : consommation facturée (dans le mois)

> La somme des AF+CF indique ce que les comptes ont été facturés dans le mois pour toute la durée où ils ont été compte "A".

> La somme des AC+CC indique ce que les comptes _ont coûté_ à l'organisation (AC + CC >= AF + CF).

> Le rapprochement de `NN+NC+NG` avec `QN` permet de comparer l'usage effectif par rapport aux quotas (de même entre `V` et `QV`).

## Archive des _tickets_ de paiement
Cette archive (un fichier CSV) contient une ligne par _ticket de paiement_ émis / reçu dans le mois. Le ticket est anonyme, il n'est pas possible d'effectuer un rapprochement avec le compte qui l'a émis.

Elle permet au Comptable d'analyser / totaliser ce qu'il a reçu en paiement, du moins ce qu'il a enregistré avoir reçu.

Colonnes:

    IDS,TKT,DG,DR,MA,MC,REFA,REFC

- `IDS` : identifiant du ticket commençant par `aamm` (l'année et le mois) suivi de 8 chiffres aléatoires.
- `TKT` : code du ticket.
- `DG` : date de génération sous la forme `aaaammjj`
- `DR`: date de réception / enregistrement sous la forme `aaaammjj`. Si 0 le ticket est _en attente_.
- `MA`: montant en `c` déclaré émis par le compte A.
- `MC` : montant en `c` déclaré reçu par le Comptable.
- `REFA` : référence facultative écrite par le compte A à l'émission.
- `REFC` : référence facultative écrite par le Comptable à la réception.
