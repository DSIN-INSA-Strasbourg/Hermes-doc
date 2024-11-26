---
title: Workflow
weight: 1
---

hermes-server

- 1\. charge son cache local
- 2\. vérifie si son schéma de données a changé depuis la dernière exécution, et émet les événements supprimés résultants (le cas échéant), ainsi que le nouveau schéma de données
- 3\. récupère toutes les données requises par son modèle de données depuis la ou les sources de données
  - 3.1. applique les contraintes de fusion
  - 3.2. fusionne les données
  - 3.3. remplace les incohérences et les conflits de fusion par les valeurs en cache
  - 3.4. applique les contraintes d'intégrité
- 4\. génère un différentiel entre son cache et les données distantes récupérées
- 5\. boucle sur chaque type de différence : ajout, modification, suppression
  - 5.1. pour chaque type de différence, boucle sur chaque type de données selon leur ordre de déclaration dans le modèle de données, à l'exception du type de différence supprimé, pour lequel il s'agit de l'ordre de déclaration inverse
    - 5.1.1. boucle sur chaque différence du type de données actuel
      - 5.1.1.1. génère l'événement correspondant
      - 5.1.1.2. émet l'événement sur le bus de messages
      - 5.1.1.3. si l'événement a été émis avec succès :
        - 5.1.1.3.1. exécute l'action `commit_one` du modèle de données le cas échéant
        - 5.1.1.3.2. met à jour le cache pour refléter la nouvelle valeur de l'élément affecté par l'événement
- 6\. une fois que tous les événements ont été émis
  - 6.1. exécute l'action `commit_all` du modèle de données le cas échéant
  - 6.2. enregistre le cache sur le disque
- 7\. attend `updateInterval` et recommence à partir de l'étape `3.` si l'application n'a pas été invitée à s'arrêter

Si une exception est levée à l'étape `2.`, cette étape est recommencée jusqu'à ce qu'elle réussisse.

Si une exception est levée aux étapes `3.` à `7.`, le cache est enregistré sur le disque et le serveur recommence à partir de l'étape `3.`.
