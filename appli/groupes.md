---
layout: page
title: Les groupes et leurs notes
---

Un avatar peut créer un **groupe** dont il sera le premier membre _actif_ et y aura un pouvoir _d'animateur_. Un groupe a,
- un identifiant interne et **une carte de visite** (comme un avatar).
- **un chat partagé par les membres du groupe**, les membres sont des _avatars_.
- **des notes partagées entre les membres** qui peuvent les lire et les éditer.

Un avatar connu dans un groupe peut avoir plusieurs états successifs:
- **simple contact**: il a été inscrit comme _contact_ du groupe, le sait et peut voir la _carte de visite_ du groupe. 
  - Il peut se supprimer lui-même des contacts du groupe, le cas échéant en indiquant ne plus jamais vouloir être contacté par ce groupe. 
  - Les autres membres du groupe voit sa _carte de visite_ et son statut de _simple contact_.
- **contact invité**: un membre **animateur** a invité le contact à devenir membre actif. L'avatar invité voit cette invitation:
  - s'il l'accepte il deviendra membre actif.
  - sinon il retournera à l'état de simple contact. 
  - nul ne devient membre actif à son insu.
- **membre actif**: il peut participer à la vie du groupe.
- **membre animateur**: c'est un _membre actif_ ayant reçu pouvoir d'animation.

## Accès aux membres et / ou aux notes
Un membre actif _peut_ recevoir lors de son invitation des _droits_:
- **droit d'accès aux autres membres** et au _chat_ (ou non),
- **droit d'accès aux notes** en _lecture_ ou en _lecture et écriture_ (ou pas du tout).

Lors de son invitation il peut aussi recevoir le **pouvoir d'animation**. S'il ne l'a pas, un membre _animateur_ peut lui conférer ce pouvoir (mais ne pourra plus lui enlever).

> **Certains groupes peuvent être créés à la seule fin d'être un répertoire de contacts** cooptés par affinité avec possibilités de _chat_. Personne n'y lit / écrit de notes.

> **Certains groupes peuvent être créés afin de partager des notes _anonymes_ de discussion**: par exemple un animateur est seul à avoir droit d'accès aux membres, à les connaître: les notes sont de facto anonymes pour les autres membres.

En général les groupes sont créés avec le double objectif de réunir des avatars qui se connaissent mutuellement, échangent sur le chat et partagent des notes.

### Tout membre actif peut attacher un commentaire personnel et ses propres _hashtags_ à un groupe
La recherche d'un groupe quand on est membre de beaucoup de groupes en est facilitée. Personne d'autre n'en a connaissance, son commentaire et ses _hashtags_ restent strictement privés.

## Modes d'invitation: _un seul animateur_ ou _unanime_
- **Mode _un seul animateur_**. Il suffit qu'un _animateur_ le décide pour qu'un _contact du groupe_ soit invité.
- **Mode _unanime_**. Il faut que tous les _animateurs_ aient invité un _contact du groupe_ pour que son invitation soit validée.

> Imaginons un _couple_ où tous deux sont à égalité _animateur_. Si le mode _unanime_ n'existait pas, l'un des deux pourrait inviter une tierce personne dans le couple, laquelle pourrait voir toutes les notes et le chat du couple, bref le groupe deviendrait un _trouple_ sans que l'autre n'ait accepté cette évolution.

Pour un groupe en mode _unanime_ passe au mode d'invitation par _un seul animateur_, il faut que tous les animateurs aient _voté_ ce changement.

A l'inverse un groupe en mode d'invitation par _un seul animateur_ peut passer en mode _unanime_ sur demande d'un seul animateur.

## Processus d'invitation
**La première étape consiste à inscrire comme _simple contact du groupe_** l'un de ses propres contacts ce qui est possible pour tous les membres ayant accès aux membres du groupe.
- les membres du groupe peuvent ainsi voir la _carte de visite_ de ce nouveau contact du groupe (mais pas ouvrir un _chat_ avec lui).
- uns discussion peut s'engager sur l'opportunité d'inviter ce contact, sur le _chat du groupe_, dans les notes, etc.

A ce moment, le _simple contact_ peut être _effacé / oublié_ par un animateur.
- il peut même être inscrit en liste noire afin de prévenir sa réinscription ultérieure comme _simple contact_.

**La seconde étape est l'invitation par un animateur du _simple contact_** en lui fixant ses conditions de participation au groupe:
- accès ou non aux membres et au chat du groupe,
- accès ou non aux notes, en lecture ou lecture / écriture,
- droit d'animation. Le droit d'animation impose le droit d'accès aux membres.

Si le mode d'invitation est _unanime_ tous les animateurs doivent valider cette invitation:
- si l'un deux changent les conditions, ceci invalide les invitations des autres.
- si l'un deux le souhaite l'invitation est annulée.
- tant que l'invitation n'a pas été validée par tous les animateurs, le contact est en état **pré-invité**.

Tant que l'invitation n'a été ni acceptée, ni refusée, elle peut être annulée par un animateur.

**La dernière étape est l'acceptation ou le refus de l'invitation par l'invité**:
- en cas d'acceptation il devient membre _actif_.
- en cas de refus, 
  - soit il redevient _simple contact_, 
  - soit il est _oublié_ du groupe (n'est plus un contact du groupe),
  - soit il est oublié définitivement et inscrit en liste noire pour éviter d'être ultérieurement réinscrit comme contact du groupe.

## Résiliation d'un membre
Un membre peut _s'auto-résilier_ du groupe, redevenant simple contact, oublié ou oublié définitivement.

Un animateur peut résilier un membre actif _non animateur_, le rendant simple contact, oublié ou oublié définitivement.

## Quelques règles
Seul um membre actif **ayant pouvoir d'animation** peut,
- donner / retirer le droit d'accès aux autres membres et au _chat_ à un membre actif donné,
- donner / retirer le droit d'accès aux notes à un membre actif non animateur (en lecture ou lecture / écriture), 
- donner un pouvoir d'animation à un membre actif qui ne l'a pas,
- inviter un _simple contact_ à devenir membre actif, avec ou sans droit accès aux autres membres et au _chat_, avec ou sans droit d'accès aux notes, avec ou sans pouvoir d'animateur,
- _résilier_ un simple contact ou un membre actif non animateur qui n'apparaîtra plus dans le groupe.

**Tout membre actif** peut,
- s'il a droit d'accès aux membres, inscrire comme _simple contact_ un de ses contacts,
- décider de ne plus utiliser ses droits d'accès aux membres et / ou aux notes, puis décider de les utiliser à nouveau.
- décider de redevenir _simple contact_, voire d'être _oublié_ par le groupe (ne figurant plus comme _simple contact_).

> Un _membre actif animateur_ ne peut pas changer les droits et pouvoir d'animation d'un autre _animateur_ (sauf à lui-même).

> Un _animateur_ peut _résilier_ un membre actif non animateur indésirable ou lui retirer ses droits d'accès aux autres membres et aux notes, donc de facto lui interdire tout accès au groupe.

Un membre actif d'un groupe ayant accès aux membres voit les autres membres actifs:
- s'il est **animateur** tous sans restriction.
- s'il n'est pas animateur seulement ceux ayant accès aux membres.

> Lorsqu'un membre actif se restreint à ne plus voir les autres membres actifs, il devient invisible aux autres membres actifs **sauf les animateurs**.

Quand un membre actif en voit un autre, il l'a comme _contact temporaire_ et en connaît sa carte de visite: il _peut_ ouvrir un chat avec lui. Tout membre ayant formellement accepté une invitation et le droit de voir les autres, a accepté la conséquence d'être visible par les autres.

> Le fait d'établir un _chat_ avec un membre du groupe en fait un contact _permanent_, même si le groupe est ultérieurement dissous ou que le membre cesse d'y être actif.

> Un groupe disparaît de lui-même dès lors qu'il n'a plus de membres actifs.

> Une résiliation ou auto-résiliation peut indiquer une inscription en _liste noire_: l'avatar ne pourra plus ni figurer comme contact, ni être inviter dans ce groupe.

# Notes d'un groupe
- elles sont cryptées par la clé aléatoire spécifique au groupe qui a été transmise à chaque membre lors de son invitation au groupe.
- hormis les membres actifs du groupe ayant droit d'accès aux notes, personne ne peut accéder aux notes du groupe.
- quand un nouveau membre accepte une invitation au groupe avec droits d'accès aux notes, il a immédiatement accès à toutes les notes existantes du groupe. S'il redevient _simple contact_ ou perd son droit d'accès aux notes (de par sa volonté ou celle d'un _animateur_), il n'a plus accès à aucune de celles-ci. Ceci allège ses sessions.
- pour écrire / modifier / supprimer une note du groupe, il faut avoir le droit d'accès en écriture aux notes.
- chaque note est signée par la succession des membres qui y sont intervenu.

## Tout membre ayant accès aux notes peut attacher ses propres _hashtags_ à chaque note du groupe
- Le filtrage des notes par _hashtags_ s'effectue tous groupes confondus. 
- Les autres membres ne savant pas quels sont ces _hashtags_.

## Une note de groupe peut être rattachée à une autre note parent du groupe
Ceci fait apparaître visuellement à l'écran une hiérarchie.

**Un avatar peut attacher une note personnelle à une note de groupe** pour la compléter / commenter: toutefois il sera seul à la voir (puisqu'elle est _personnelle_).

## Exclusivité d'écriture à une note
Il est intéressant que certaines notes ne puissent être écrites / mises à jour que par un seul membre.

**On peut conférer à un membre le droit exclusif d'écrire une note donnée:**
- un animateur peut créer / changer / retirer cette exclusivité.
- le membre ayant l'exclusivité peut se la retirer et en confier l'exclusivité à un autre (ou à personne).

## Membre _hébergeur_ d'un groupe
_L'hébergeur du groupe_ est un membre qui s'est dévoué pour supporter les coûts d'abonnement de stockage (nombres de notes et volume des fichiers) des notes du groupe.
- il fixe des maximum à ne pas dépasser afin de protéger sa facture et l'espace dont il dispose pour ses autres besoins,
- il peut cesser d'héberger le groupe, le groupe **n'est plus hébergé**.

Un groupe non hébergé s'auto-détruit trois mois après le jour où il a perdu son hébergeur. D'ici là,
- le nombre de notes ne peut plus augmenter,
- le volume des fichiers ne peut que décroître,
- des membres du groupe peuvent se déclarer _hébergeur_.

#### Quelques règles:
- quand un groupe n'a plus d'hébergeur, n'importe quel membre peut se déclarer _hébergeur_ et fixer ses limites.
- quand un groupe a un hébergeur **non animateur**: n'importe quel membre peut se déclarer _hébergeur_ à sa place.
- quand un groupe a un hébergeur **animateur**: seul un autre animateur peut se déclarer _hébergeur_ à sa place.
