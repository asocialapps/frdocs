---
layout: page
title: Spécifications
---

# Architecture, développement, déploiements

[Architecture Générale](tech/architecturegen.html)
- Présentation des composants majeurs (application Web, serveurs / Cloud Functions, utilitaires ...) constituant _l'application_. (17 pages).

[Le service OP des "opérations" d'accès à la base](tech/sfd_op-db-st.html)
- Spécifications Fonctionnelles Détaillées du service OP des opérations de mise à jour / consultation de la base de données et du _storage_ des fichiers. (78 pages).

[Le service PUBSUB de "notification" des mises à jour](tech/sfd_pubsub.html)
- Spécifications Fonctionnelles Détaillées du service PUBSUB.
- Le service PUBSUB gère les _souscriptions_ des sessions des application Web en cours et la _publication_ (notifications) vers ces sessions des avis de modification des documents de leur périmètre à la fin de chaque opération. (4 pages).

[Design des "serveurs", leur API](tech/apiserveur.html)
- Le design technique et l'API des _serveurs_. (35 pages).

[Les environnements de développement](tech/developpement.html)
- Studio, langages, outils de tes locaux, modules utilisés. (35 pages).

[Les déploiements des composants de l'application](tech/deploiements.html)
- L'application comporte plusieurs _composants_ (application Web, serveurs / Cloud functions, utilitaires ...), chacun a son processus spécifique de déploiement.
