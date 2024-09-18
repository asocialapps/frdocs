---
layout: page
title: Modes de connexion synchronisé, avion, incognito
---

Pour se connecter à son compte, le titulaire d'un compte choisit sous quel **mode** sa session va s'exécuter: _synchronisé_, _avion_ ou _incognito_.

## Mode "normal" _synchronisé_ 
C'est le mode préférentiel où toutes les données du périmètre d'un compte sont stockées dans une micro base locale cryptée dans le navigateur: elle est remise à niveau depuis le serveur central à la connexion d'une nouvelle session.

Durant une session la micro base locale du compte est maintenue à jour, y compris lorsque d'autres sessions s'exécutent en parallèle sur d'autres navigateurs et mettent à jour les données du compte : par exemple quand une note de groupe est mise à jour par un autre membre du groupe.

Une connexion ultérieure d'un compte dans le même navigateur après une session synchronisée est rapide et économique: l'essentiel des données étant déjà dans le navigateur, seules les _mises à jour_ sont tirées du serveur central.

## Mode _avion_
Pour que ce mode fonctionne il faut qu'une session antérieure en mode _synchronisé_ ait été exécutée dans ce navigateur pour le compte. A la connexion le titulaire du compte y voit l'état dans lequel étaient ses données à la fin de sa dernière session synchronisée dans ce navigateur.

**L'application ne fonctionne qu'en lecture**, aucune mise à jour n'est possible. Aucun accès à Internet n'est effectué, ce qui est précieux _en avion_ ou dans les _zones blanches_ ou quand l'Internet est suspecté d'avoir de grandes oreilles indiscrètes : certes tout est crypté et illisible mais en mode avion personne ne peut même savoir que l'application a été ouverte, l'appareil peut être physiquement isolé du Net.

> Toutefois on peut enregistrer des textes et des fichiers (des photos par exemple) localement (au _brouillon_): ils pourront être utilisés dans une future session _synchronisée_ (voir ci-après).

En mode avion les fichiers attachés aux notes ne sont pas accessibles, **sauf** ceux qui ont été déclarés devoir l'être. Cette déclaration pour un compte s'effectue fichier par fichier pour chaque navigateur et ils sont mis à jour à l'ouverture de chaque session en mode _synchronisé_ (puis en cours de session). (voir ci-après)

> On peut couper le réseau (le mode _avion_ sur un mobile), de façon à ce que l'ouverture de la page de l'application ne cherche même pas à vérifier si une version plus récente est disponible.

## Mode _incognito_
**Aucun stockage local n'est utilisé, toutes les données viennent du serveur central**, l'initialisation de la session est plus longue qu'en mode synchronisé. Aucune trace n'est laissée sur l'appareil (utile au cyber-café ou sur le mobile d'un.e ami.e) : certes les traces en question auraient été inutilisables car cryptées, mais il n'est pas poli d'encombrer la mémoire d'un appareil qu'on vous a prêté.

> On peut ouvrir l'application dans une _fenêtre privée_ du navigateur, ainsi même le texte de la page de l'application sera effacé en fermant la fenêtre.

> **En utilisant des sessions synchronisées sur plusieurs appareils, on a autant de copies synchronisées de ses notes et chats sur chacun de ceux-ci**, et chacun peut être utilisé en mode avion. Les copies ne sont pas exactement les mêmes, les _photographies_ de l'état des données du compte ne pouvant pas être effectuées exactement à la même seconde.

> **La page de l'application invoquée depuis le navigateur y est mémorisée**: au prochain appel de celle-ci, comme elle est déjà présente en local, elle ne chargera rien ou juste le minimum nécessaire pour se mettre à niveau de la version logicielle la plus récente.

# Fichiers accessibles en mode _avion_

Au cours d'une session _synchronisée_ dans un navigateur, on peut voir les notes et leurs fichiers attachés. Pour chaque _fichier_ ayant un nom donné, on voit la liste de ses _révisions_, quand on a décidé d'en garder plus d'une. En général on garde la dernière, mais on peut aussi revenir à une révision antérieure quand on en garde plusieurs.

En présence d'un fichier donné, on peut:
- soit décider de ne pas l'avoir accessible en mode _avion_,
- soit d'avoir _la révision la plus récente_ accessible en mode _avion_,
- soit d'avoir une ou plusieurs révisions explicitement citées accessibles en mode _avion_.

Quand on a déclaré des fichiers accessibles en mode _avion_, ils sont chargés dans la base locale du compte dans navigateur de manière à être lisible même en l'absence de réseau. Ces fichiers sont maintenus à jour, en début de session synchronisée et en cours de ces sessions, même quand les mises à jour ont été effectuées depuis d'autres sessions.

> Remarque: les navigateurs n'ont pas de politiques très claires sur le volume que leurs bases locales peut supporter. Éviter de définir trop de fichiers volumineux.

> Remarque: la déclaration d'accessibilité en mode _avion_ est spécifique du navigateur utilisé, donc de l'appareil sur lequel il s'exécute. On peut avoir un fichier _accessible en mode avion_ sur son PC et pas sur son mobile par exemple.

> Quand un fichier est accessible en mode _avion_, il n'est pas redemandé au serveur de stckage mais lu localement, ce qui est plus rapide et évite le coût de téléchargement. Certes ces coûts sont modiques mais pour certains fichiers très fréquemment lus, ça peut devenir significatif.

### Vue des fichiers accessibles en mode _avion_
Cette vue de l'application permet,
- de gérer cette option pour chaque fichier,
- d'afficher / sauvegarder dans son répertoire de _Téléchargements_ les fichiers listés.

Cette vue est disponible en modes _synchronisé_ et _avion_ (mais pas en mode _incognito_).

# Le _presse-papier_
Cette fonctionnalité est disponible dans les trois modes.

Le _presse-papier_ garde localement des _textes_ et des _fichiers_, chacun portant un code identifiant.
- on peut ajouter un texte en le saisissant ou en le copiant / collant depuis n'importe quel endroit, en particulier depuis le texte d'une note.
- on peut aussi l'éditer.
- on peut ajouter un fichier,
  - soit en le sélectionnant depuis un des fichiers disponible sur le poste,
  - soit en _copiant_ le fichier d'une note.

Dans une note on peut _coller_ du texte, en particulier celui d'un texte du _presse-papier_.

On peut attacher comme fichier d'une note, l'un de ceux présent dans le _presse-papier_.

Le _presse-papier_ est plus ou moins persistant d'une session à la suivante:
- en mode _incognito_, il n'est pas mémorisé sur le poste, et sa durée de vie est donc celle de la session en cours.
- en modes _synchronisés_ et _avion_ , le _presse-papier_ est sauvegardé dans la base locale du compte dans le navigateur: son contenu est disponible à l'ouverture de la session suivante (_synchronisés_ ou _avion_).

### Exemple d'usage du _presse-papier_
Dans une zone blanche, on peut ouvrir une session en mode _avion_, mais pas effectuer de mise à jour.

Il est possible toutefois de _préparer_ des mises à jour:
- ouvrir le texte d'une note, le mettre dans le _presse-papier_ et l'éditer.
- prendre des photos et les enregistrer dans le _presse-papier_.

Le contenu du _presse-papier_ est crypté, inaccessible à quiconque n'est pas le compte propriétaire: même si le mobile a été volé, son contenu reste indéchiffrable.

Une fois retourné dans une zone ayant du réseau, on peut ouvrir une session en mode _synchronsé_. Le _brouillon_ de la note mise à jour et les photos mises en _presse-papier_ sont accessibles pour effectuer la véritable mise à jour.
