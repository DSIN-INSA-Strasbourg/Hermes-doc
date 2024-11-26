---
title: Fichiers cache
weight: 6
---

## _hermes-server.json

Contient l'état du serveur :

- ```py
  lastUpdate: datetime.datetime | None
  ```

  Date et heure de la dernière mise à jour.

- ```py
  errors: dict[str, dict[str, dict[str, Any]]]
  ```

  Dictionnaire contenant les erreurs actuelles, pour pouvoir notifier de tout changement.

- ```py
  exception: str | None
  ```

  Chaîne contenant la dernière trace d'exception.

## _dataschema.json

Contient le schéma de données, construit depuis le modèle de données. Ce fichier cache permet au serveur de traiter l'étape `2.` de [Workflow](/hermes/how-it-works/hermes-server/workflow/).

## *DataType*.json

Un fichier par type de données déclaré dans Datamodel, contenant le cache des données de ce type de données, sous forme de liste de dictionnaires. Chaque dictionnaire de la liste représente une entrée.
