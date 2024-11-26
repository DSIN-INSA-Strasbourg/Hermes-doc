---
title: Fonctionnalités
weight: 2
---

- Ne nécessite aucune modification du ou des modèles de données sources (*e.g.* pas besoin d'ajouter une colonne `last_updated`)
- Multi-source, avec possibilité de [fusionner](/hermes/how-it-works/hermes-server/data-merging/) ou d'[agréger](/hermes/how-it-works/hermes-server/data-aggregation/) des données, et éventuellement de définir des contraintes de fusion/agrégation
- Capable de gérer plusieurs types de données, avec des liens (*[clés étrangères](/hermes/how-it-works/hermes-client/foreign-keys/)*) entre elles, et d'appliquer des [contraintes d'intégrité](/hermes/how-it-works/hermes-server/integrity-constraints/)
- Capable de transformer des données avec des [filtres Jinja](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters) dans les fichiers de configuration : pas besoin d'éditer du code Python
- Gestion des erreurs propre, pour éviter les problèmes de synchronisation, et un mécanisme optionnel d'[auto-remédiation](/hermes/how-it-works/hermes-client/auto-remediation/) des erreurs
- Propose une [corbeille](/hermes/key-concepts/#trashbin) sur les clients pour les données supprimées
- Insensible à l'indisponibilité et aux erreurs sur chaque lien (source, bus de messages, cible)
- Facile à étendre par conception. Tous les éléments suivants sont implémentés en tant que [plugins](/development/plugins/) ([liste des plugins existants](/setup/configuration/plugins/)) :
  - Sources de données
  - Filtres d'attributs (filtres de données)
  - Clients (cibles)
  - Bus de messages
- Les modifications apportées au modèle de données sont faciles et sûres à intégrer et à propager, que ce soit sur le [serveur](/maintenance/server-datamodel-update/) ou sur les [clients](/maintenance/client-datamodel-update/)
