---
layout: page
title: Configurer le thème de l'application
---

L'application utilise 2 jeux de 16 couleurs:
- 16 couleurs pour le style _sombre_,
- 16 couleurs pour le style _clair_.

Il est possible de changer simplement ces couleurs dans le fichier de configuration de l'application. La première valeur correspond au style _sombre_ la seconde au style _clair_.

    theme: {
      primary: ['#0D47A1', '#BBDEFB'],
      secondary: ['#33691E', '#DCEDC8'],
      info: ['#82C8E8', '#0101FF'],
      accent: ['#9C27B0', '#9C27B0'],
      positive: ['#21BA45', '#21BA45'],
      negative: ['#C10015', '#C10015'],
      warning: ['#E65100', '#E65100'],
      msgbg: ['#FFF176', '#FFF176'],
      msgtc: ['#B71C1C', '#B71C1C'],
      tbptc: ['#82C8E8', '#0101FF'],
      tbstc: ['#DCEDC8', '#212121'],
      btnbg: ['#1976D2', '#1976D2'],
      btntc: ['#FFFFFF', '#FFFFFF'],
      btwbg: ['#E65100', '#E65100'],
      btwtc: ['#FFFFFF', '#FFFFFF'],
      mdtitre: ['#64B5F6', '#1565C0']
    }

`primary secondary info accent positive negative warning`

Ce sont les couleurs prédéfinies dans Quasar: `info` n'est pas utilisé (sous ce nom).

Les autres codes ont des fonctions spécifiques pour l'application:
- `msgbg msgtc`: couleur de fond et de texte pour afficher un diagnostic d'erreur / attention.
- `tbpbg tbptc`: couleur de fond et de texte des barres _primaires_.
- `tbsbg tbstc`: couleur de fond et de texte des barres _secondaires_.
- `btnbg btntc`: couleur de fond et de texte des boutons déclenchant une action _normale_.
- `btwbg btwtc`: couleur de fond et de texte des boutons déclenchant une action _potentiellement risquée_.
- `mdtitre`: couleur du texte des titres dans les textes longs, notes, cartes de visite, commentaires personnels, documentation.
