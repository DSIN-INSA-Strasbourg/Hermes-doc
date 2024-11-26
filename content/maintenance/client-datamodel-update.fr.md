---
title: Mise à jour du modèle de données client
weight: 2
---

Un modèle de données n'est pas figé dans le temps, il peut évoluer et donc être mis à jour, que ce soit depuis le serveur ou sur un ou plusieurs clients.

Chaque fois que le modèle de données est modifié sur un client, le client génère des événements locaux pour propager les modifications de données consécutives sur les cibles.
Le client peut notifier des avertissements concernant le modèle de données si certains types de données ou attributs distants sont définis dans son modèle de données mais n'existent pas dans le schéma de données actuel reçu d'hermes-server.

## Ajouter un attribut à un type de données existant

1. 👱 Ajouter l'attribut aux [modèles de données clients](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "modified" depuis le cache complet

## Supprimer un attribut d'un type de données

1. 👱 Supprimer l'attribut des [modèles de données clients](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "modified" consécutifs

## Modifier la valeur d'un attribut (en changeant son filtre Jinja ou son attribut distant de la source de données)

1. 👱 Modifier l'attribute dans les [modèles de données client](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "modified" consécutifs

## Ajouter un nouveau type de données

### Si son *hermesType* existe déjà dans le schéma de données

1. 👱 Ajouter le type de données aux [modèles de données client](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "added" depuis le cache complet

### Si son *hermesType* n'existe pas encore dans le schéma de données

1. 👱 Ajouter le type de données aux [modèles de données client](/hermes/key-concepts/#client-datamodel) afin qu'ils puissent le traiter lorsqu'il sera ajouté au [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer les clients : ⚠️ avertissement du modèle de données "*remote types don't exist in current Dataschema*"
2. 👱 Ajouter le type de données aux [modèles de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "added" pour chaque entrée du type de données ajouté
3. 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma. ✅ Plus d'avertissement de modèle de données. Traitement des événements "added" entrants

## Supprimer un type de données existant

1. 👱 Supprimer le type de données dans les [modèles de données client](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "removed" consécutifs
    - 💻 Purge des fichiers de cache locaux du type de données supprimé
