---
layout: page
title: Alertes et restrictions d'accès associées
---

### Alertes de _comptabilité des coûts_
Le décompte des coûts est susceptible de **lever des alertes** à chaque compte en fonction de sa situation personnelle.

#### Coût de calcul excédant son quota (QC): ralentissements
Les coûts de calcul relevés sur le mois en cours et le précédent sont ramenés à un _mois de 30 jours_. Il y a excès quand ce coût moyen de calcul excède le quota QC.
- les opérations sont artificiellement ralenties par un délai d'attente d'autant plus grand que l'excès est fort.
- les transferts de fichiers le sont également selon le même coefficient mais proportionnellement au volume du fichier.

> Remarque: un compte "A" étant lui-même maître de son quota QC peut éviter les ralentissements en augmentant fortement son quota QC: ce faisant il inhibe une alerte de surconsommation de calcul qui peut s'avérer coûteuse.

> Remarque: à tout instant un compte peut voir le nombre de jours estimé en solde positif en considérant la prolongation de sa situation actuelle (mêmes quotas, même consommation moyenne de calcul).

#### Nombre de documents excédant son quota (QN): blocage de l'ajout de documents
Le compte ne peut pas créer de nouvelles notes, de chats ou participer à de nouveaux groupes. Il est ainsi inviter à,
- supprimer des notes,
- déclarer des chats _indésirables_,
- se résilier de certains groupes,
- OU ... demander une augmentation de son quota QN au Comptable ou à un de ses délégués si c'est un compte "O", procéder lui-même à cette augmentation si c'est un compte "A".

#### Volume de fichiers attachés aux notes excédant son quota (QV): blocage du volume
Le compte ne peut pas ajouter de fichiers ou remplacer un fichier par un autre de taille supérieure. Il est ainsi inviter à,
- supprimer des fichiers et/ou des révisions de fichiers,
- remplacer les gros fichiers par des révisions plus compressés,
- OU ... demander une augmentation de son quota QV au Comptable ou à un de ses délégués si c'est un compte "O", procéder lui-même à cette augmentation si c'est un compte "A".

#### Solde négatif: le compte est mis en ACCÈS RESTREINT
Le compte ne peut plus ni lire ses données, ni les mettre à jour. Toutefois il conserve les possibilités de:
- visualiser ses alertes et sa comptabilité,
- effectuer des versements et les enregistrer afin que le Comptable les créditent,
- discuter  de sa situation sur les _chats d'urgence_ avec le Comptable et ses _délégués_ pour un compte "O, voire de bénéficier d'un _don_ (à rembourser plus tard le cas échéant).

## Alerte de l'Administrateur Technique à l'organisation
**Cette alerte porte toujours un message d'information** (_arrêt programmé ..._).

Elle _peut_, soit ne porter aucune restriction, soit porter l'une de celles-ci:
- **Espace figé**. Aucune écriture ne peut être faite, typiquement afin de procéder à une opération technique d'export, verrouillage d'une archive d'un espace ... mais peut aussi être une mesure de rétorsion.
- **Espace clos**. L'Administrateur Technique a effacé les données de l'espace dont il ne subsiste plus que cette alerte dont le texte donne la raison et le cas échéant indique si l'espace est accessible à une autre URL.
- **Connexions bloquées à partir d'une date d**: les comptes ne seront pas détruits à cette date mais plus aucune connexion ne pourra être faite

### Alerte du _Comptable_ ou de ses _délégués_ aux comptes "O"
Le Comptable ou ses délégués peuvent inscrire une _alerte_:
- adressée à TOUS les comptes d'un _partition_,
- adressée à UN compte "O" spécifique.

Chaque compte "O" est en conséquence potentiellement la cible de deux alertes.

Une telle alerte **porte un message d'information**, et peut être associé ou non à une restriction.

#### Accès en LECTURE SEULEMENT
Le compte **ne peut que consulter les données** (comme en mode _avion_).

#### ACCÈS RESTREINT
Le compte ne peut plus ni lire ses données, ni les mettre à jour. 

Dans tous les cas il conserve toutefois les possibilités de:
- visualiser ses alertes et sa comptabilité,
- effectuer des versements et les enregistrer afin que le Comptable les créditent,
- discuter  de sa situation sur les _chats d'urgence_ avec le Comptable et ses _délégués_ pour un compte "O", voire de bénéficier d'un _don_ (à rembourser plus tard le cas échéant).

## Date limite de vie d'un compte
Cette date, toujours le dernier jour d'un mois, signifie que le jour suivant le compte sera irrémédiablement détruit sans possibilité de récupération.

Cette date est fixée lorsqu'un compte se connecte: en conséquence le compte en est immédiatement informé.

Quand le compte **N'EST PAS en ACCÈS RESTREINT** elle est fixée à 12 mois plus tard. En d'autres termes le compte a toujours un an de vie après sa dernière connexion.

Quand le compte **EST en ACCÈS RESTREINT** elle est fixée à 6 mois après que le compte a basculé en ACCÈS RESTREINT.

A chaque connexion un compte voit la date limite de vie de son compte et le nombre de jours à partir du jour courant. Si ce nombre est faible (inférieur à 30), l'alerte est **rouge**.

> Remarque: un compte n'a de facto que 6 mois de vie assurée après la dernière connexion où il a constaté de na pas être en ACCÈS RESTREINT. Un compte "A" est maître de cette durée en s'assurant que son compte restera créditeur. Un compte "O" peut à l'inverse faire l'objet d'un ACCÈS RESTREINT imposé par le Comptable ou un de ses délégués, et ce en dehors de son contrôle.

## "12 mois", ou plus, ou moins
Le nombre 12 mois ci-dessus est celui _par défaut_ mais le Comptable peut le changer pour d'autres valeurs 3, 6, 12, 18, 24.

> Plus cette durée est longue plus le risque est grand qu'un compte ait une fin de vie longue en ACCÈS RESTREINT se terminant par une _ardoise_, un solde négatif non couvert.

[Maîtrise des coûts d'hébergement de l'application](../tech/coutshebergements.html).
