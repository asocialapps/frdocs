---
layout: page
title: La gestion des espaces par l'administrateur technique
---

**L'administrateur technique** d'un site peut y héberger techniquement plusieurs **espaces** dans la même base et le même _storage_.

Les utilisateurs n'ont aucune perception des autres espaces hébergés par le même serveur technique.
- dans la base de données, les informations sont partitionnées.
- dans l'espace de stockage des fichiers, des sous-espaces sont séparés par nom de l'organisation.

> Certaines organisations peuvent souhaiter avoir plus d'un espace pour elle: par exemple un espace de _production_, un espace de _démonstration / training_, un espace _archive récente_ ...

L'administrateur technique a la possibilité d'ouvrir _instantanément_ un nouvel espace pour une organisation en faisant la demande. 
- Le Comptable et l'administrateur technique se sont mis d'accord sur le volume utilisable et la participation aux frais d'hébergement.
- Cette ouverture crée une phrase de _sponsoring_ à destination du Comptable de l'organisation, 
- Le Comptable créé son compte en utilisant cette phrase de _sponsoring_ et en fixant sa phrase secrète.

## Notifications de l'administrateur technique
L'administrateur technique peut émettre 3 notifications à l'espace d'une organisation.

#### Informative sans restriction
Ce peut-être par exemple l'annonce d'une indisponibilité temporaire planifiée, mais aussi l'annonce d'une fermeture définitive.

#### Annonce de _figeage_
L'espace est strictement en lecture, plus aucune écriture n'est possible.

Quand ce n'est pas une mesure de rétorsion, c'est afin de permettre une exportation des données dans un état cohérent.

#### Annonce de _clôture_
Le message explique que l'espace n'est plus disponible, que ses données ont été supprimées. Seul ce message subsiste: si l'espace a été réinstallé et est joignable sur une nouvelle URL, c'est l'occasion de l'indiquer.

## Export d'un espace
L'exportation d'un espace se fait sur un espace _figé_ et concerne deux aspects: la base de données et l'espace de fichiers.

L'administrateur peut transférer toutes les données de la base de données sur une autre base, en général _locale_ au poste de l'administrateur. A cette occasion le _code_ de l'organisation peut changer (comme d'ailleurs la clé de cryptage des données en base).

L'administrateur peut transférer tous les fichiers sur une autre _Storage_ en général _local_ au poste de l'administrateur. A cette occasion le _code_ de l'organisation peut changer (comme d'ailleurs la clé de cryptage des identifiants des fichiers -leur _path_, pas leur contenu).

En toute rigueur la base de données et le _storage_ cible **peuvent** ne pas être sur des supports locaux, mais des restrictions techniques peuvent rendre l'opération impossible (par exemple transfert depuis un compte _Google_ vers un autre compte _Google_).

### Changer de prestataire
Dans tous les cas de figure l'exportation peut s'effectuer vers un support technique _local_ que tous les prestataires ont configuré. Il suffit alors que le prestataire _origine_ transfère au prestataire _cible_, 
- un fichier contenant la base de données accompagnée de la clé de cryptage choisie pour le transfert,
- un _folder_ représentant l'espace de fichiers, où ceux-ci sont cryptés par les clés des comptes (pas par une clé technique de l'administrateur).

> La partie _base de données_ ne peut pas avoir un volume considérable. Il n'en n'est pas de même pour l'espace de fichiers, qui lui peut être énorme (d'où l'intérêt de ne pas l'exporter en _local_)

## Purge d'un espace
L'opération consiste à supprimer le sous-ensemble de la base de données relative à l'espace à supprimer et le _folder_ racine dans le _storage_.

> **Il n'existe aucune raison technique de bonne foi qui interdise à un prestataire l'exportation d'un espace** dans un format tel qu'un autre prestataire ne puisse le reprendre.
