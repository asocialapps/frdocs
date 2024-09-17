
# En savoir plus ...


## Quelques compteurs statistiques d'usage des volumes d'un compte
Les volumes V1 sont ceux des textes des notes, les volumes V2 sont ceux des fichiers attachés aux notes.

Pour un compte, les compteurs suivants reflètent l'usage de l'espace:
- quotas actuels en volumes V1 et V2 attribués par le Comptable au compte.
- volumes V1 et V2 effectivement utilisés pour les notes.
- volumes moyens V1 et V2 pour le mois en cours.
- volumes V2 des fichiers transférés :
  - cumul de la journée pour chacun des sept derniers jours,
  - cumul sur le mois courant.
- **pour les 12 derniers mois**,
  - les quotas au dernier jour du mois.
  - la moyenne des volumes journaliers du mois,
  - le volume total des transferts de fichiers dans le mois.

> Un compte _non sponsor_ n'a accès qu'à ses propres statistiques. Le Comptable a accès à toutes celles des comptes et un sponsor à celles des comptes de sa tribu. Cette connaissance leutr permet par exemple d'apprécier :
>- quand un sponsor de tribu réclame des quotas supplémentaires, comment le volume est effectivement alloué et utilisé par les comptes,
>- quand un compte se plaint que son sponsor ne lui donne pas de quotas supplémentaires, ce qu'il utilise réellement de ses quotas.

## Unités de volume
Les quotas sont donnés en nombre d'unités ci-dessous :
- pour V1 : 0,25 MB
- pour V2 : 25 MB

Les quotas s'étagent de 1 à 255. Certains quotas ont un code symbolique. L'ordre de grandeur du coût mensuel pour l'hébergeur est donné ci-après pour information en centimes d'euro :
- (1) - XXS - 0,25 MB / 25 MB - 0,09c
- (4) - XS - 1 MB / 100 MB - 0,35c
- (8) - SM - 2 MB / 200 MB - 0,70c
- (16) - MD - 4 MB / 400 MB - 1,40c
- (32) - LG - 8 MB / 0,8GB - 2,80c
- (64) - XL - 16 MB / 1,6GB - 5,60c
- (128) - XXL - 32 MB / 3,2GB - 11,20c
- (255) - MAX - 64 MB / 6,4GB - 22,40c

Le code _numérique_ d'un quota va de 0 à 255 : c'est le facteur multiplicateur du forfait le plus petit (0,25MB / 25MB). Les codes symboliques sont enregistrés dans la configuration de l'hébergement et peuvent être ajoutés / modifiés sans affecter les données déjà enregistrées.
