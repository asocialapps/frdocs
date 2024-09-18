---
layout: page
title: Notifications et restrictions d'accès des comptes
---

Une _notification_ est un message suffisamment important pour que sa présence soit signalée par une icône dans la barre d'entête de l'écran de l'application.

Une _notification_ **peut être porteuse d'une restriction d'accès**: quand une session a une ou des restrictions d'accès, ses actions sont plus ou moins limitées.

### Notification de _l'administrateur technique_ à l'espace de l'organisation
**Elle peut n'être une simple information** (_arrêt programmé ..._) sans restriction mais peut aussi fixer l'une de ces deux restrictions:
- **Espace figé**. Aucune écriture ne peut être faite, typiquement afin de procéder à une opération technique d'export, verrouillage d'une archive d'un espace ... mais peut aussi être une mesure de rétorsion.
- **Espace clos**. L'administrateur technique a effacé les données de l'espace: il ne subsiste plus que cette notification dont le texte donne la raison et le cas échéant indique si l'espace est accessible à une autre adresse.

### Notification du _Comptable_ ou de ses _délégués_ pour les comptes "O"
Le Comptable ou ses délégués peuvent inscrire une _notification_:
- adressée à tous les comptes d'un _partition_,
- adressée à un compte "O" spécifique.

Une notification **peut ne porter qu'un message d'information**, sans restriction.

Elle **peut aussi porter une des deux restrictions**: _lecture seule_ et _accès minimal_.

#### Lecture seule
En lecture seule une session **ne peut que consulter les données** (comme en mode _avion_).
- les échanges sont toutefois possibles sur les _chats d'urgence_ avec le Comptable et les _délégués de sa partition_.

#### Accès minimal
En accès minimal une session **ne peut plus ni lire ni mettre à jour** ses données.
- les échanges sont toutefois possibles sur les _chats d'urgence_ avec le Comptable et pour un compte "O" les _délégués de sa partition_.

**Les connexions du compte ne le maintiennent plus en vie**: au plus tard dans un an, si cette restriction n'est pas levée, le compte disparaîtra.

Une telle restriction pour un compte "O" est causée par:
- déclaration explicite du Comptable ou un de ses délégués.
- automatiquement quand la consommation mensuelle moyenne dépasse la limite fixée.

#### Volume en réduction
Cette restriction bloque toutes les actions menant à une augmentation de volume:
- _création_ d'une note, d'un chat, ou acceptation d'une invitation à un groupe,
- _remplacement_ d'un fichier attaché à une note par un autre plus important.

Cette restriction est **automatique**, causée par le dépassement des limites d'abonnements (nombre de documents, volume des fichiers).

### Restriction _accès minimal_ pour excès de consommation d'un compte "A"
En accès minimal une session ne peut plus ni lire ni mettre à jour ses données, toutefois:
- les échanges sont possibles sur les _chats d'urgence_ avec le Comptable.
- les opérations de crédit / gestion des volumes maximaux restent autorisées.

Elle est **automatique** lors de la détection d'un solde négatif.

> **En cas de _restriction_ les connexions des comptes ne les maintiennent plus en vie**: au plus tard dans un an, si cette restriction n'est pas levée, les comptes disparaîtront.

[Maîtrise des coûts d'hébergement de l'application](../tech/coutshebergements.html).
