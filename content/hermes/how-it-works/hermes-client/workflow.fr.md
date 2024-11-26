---
title: Workflow
weight: 1
---

hermes-client

- 1\. charge son modèle de données à partir du fichier de configuration
- 2\. s'il existe, charge le modèle de données précédent depuis le cache
- 3\. le cas échéant, notifie les avertissements du modèle de données : type distant et attributs distants présents dans le modèle de données, mais pas dans le schéma de données
- 4\. si un schéma distant existe dans le cache, charge la file d'erreurs depuis le cache
- 5\. si le client n'a pas encore été initialisé (aucune séquence initSync complète n'a encore été traitée) :
  - 5.1. traite la séquence initSync, si une séquence initSync complète est disponible sur le bus de messages
  - 5.2. redémarre à partir de l'étape `5.`
- 6\. si le client a déjà été initialisé (une séquence initSync complète a déjà été traitée) :
  - 6.1. s'il s'agit de la première itération de la boucle (l'étape `7.` n'a jamais été atteinte) :
    - 6.1.1. si le modèle de données dans la configuration diffère de celui mis en cache, traite la mise à jour du modèle de données :
      - 6.1.1.1 génère les événements de suppression pour toutes les entrées des types de données supprimés, les traite et purge les fichiers de cache de ces types de données
      - 6.1.1.2 génère un différentiel entre les données mises en cache construites sur le modèle de données précédent et les mêmes données converties au nouveau modèle de données, puis génère les événements correspondants et les traite
  - 6.2. si `errorQueue_retryInterval` est écoulé depuis la dernière tentative, réessaye de traiter les événements présents dans la file d'erreurs
  - 6.3. si `trashbin_purgeInterval` est écoulé depuis la dernière tentative, purge les objets expirés de la corbeille
  - 6.4. boucle sur tous les événements disponibles sur le bus de messages et traite chacun d'eux pour appeler sa méthode de gestion des événements correspondante lorsqu'elle existe dans le plugin client
- 7\. lorsqu'au moins un événement a été traité ou si l'application a été invitée à s'arrêter :
  - 7.1. enregistre les fichiers cache de la file d'erreurs, de l'application, des données
  - 7.2. appelle la méthode de gestion de l'événement spécial `onSave` lorsqu'elle existe dans le plugin client
  - 7.3. notifie tout changement dans la file d'erreurs
- 8\. redémarre à partir de l'étape `5.` si l'application n'a pas été invitée à s'arrêter

Si une exception est levée à l'étape `6.1.1`, elle sera considérée comme une erreur fatale, notifiée et le client s'arrêtera.

Si une exception est levée aux étapes `5.` à `6.`, elle est notifiée, son événement est ajouté à la file d'erreurs et le client redémarre à partir de l'étape `7.`.
