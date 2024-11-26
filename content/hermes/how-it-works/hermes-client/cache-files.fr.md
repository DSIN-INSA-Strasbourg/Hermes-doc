---
title: Fichiers cache
weight: 5
---

## _hermes-client-*name*.json

Contient l'état du client :

- ```py
  queueErrors: dict[str, str]
  ```

  Dictionnaire contenant l'ensemble des messages d'erreurs des objets en file d'erreurs, pour pouvoir notifier de tout changement.

- ```py
  datamodelWarnings: dict[str, dict[str, dict[str, Any]]]
  ```

  Dictionnaire contenant les avertissements actuels du modèle de données, pour pouvoir notifier de tout changement.

- ```py
  exception: str | None
  ```

  Chaîne contenant la dernière trace d'exception.

- ```py
  initstartoffset: Any | None
  ```

  Contient l'offset du premier message de la séquence initSync sur le bus de messages.

- ```py
  initstopoffset: Any | None
  ```

  Contient l'offset du dernier message de la séquence initSync sur le bus de messages.

- ```py
  nextoffset: Any | None
  ```

  Contient l'offset du prochain message à traiter sur le bus de messages.

## \_hermesconfig.json

Cache de la configuration précédente, utilisé pour pouvoir construire le modèle de données précédent et pour générer les expressions Jinja avec les plugins d'attributs.

## \_dataschema.json

Cache du dernier schéma de données, reçu d'hermes-server.

## \_errorqueue.json

Cache de la file d'erreurs.

## *RemoteDataType*.json

Un fichier par type de données distantes, contenant toutes les entrées distantes, telles qu'elles ont été traitées par le client sans générer d'erreur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [*RemoteDataType*_complete\_\_.json](#remotedatatype_complete__json)

## *RemoteDataType*_complete\_\_.json

Un fichier par type de données distantes, contenant toutes les entrées distantes, telles qu'elles sont sur le serveur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [*RemoteDataType*.json](#remotedatatypejson)

## trashbin_*RemoteDataType*.json

Uniquement si la corbeille est activée. Un fichier par type de données distant, contenant toutes les entrées distantes qui se trouvent dans la corbeille, telles qu'elles ont été traitées par le client sans générer d'erreur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [trashbin_*RemoteDataType*_complete\_\_.json](#trashbin_remotedatatype_complete__json)

## trashbin_*RemoteDataType*_complete\_\_.json

Uniquement si la corbeille est activée. Un fichier par type de données distant, contenant toutes les entrées distantes qui se trouvent dans la corbeille, telles qu'elles sont sur le serveur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [trashbin_*RemoteDataType*.json](#trashbin_remotedatatypejson)

## \_\_*LocalDataType*.json

Un fichier par type de données local, contenant toutes les entrées locales, telles qu'elles ont été traitées par le client sans générer d'erreur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [\_\_*LocalDataType*_complete\_\_.json](#__localdatatype_complete__json)

## \_\_*LocalDataType*_complete\_\_.json

Un fichier par type de données local, contenant toutes les entrées locales, telles qu'elles sont sur le serveur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [\_\_*LocalDataType*.json](#__localdatatypejson)

## \_\_trashbin_*LocalDataType*.json

Uniquement si la corbeille est activée. Un fichier par type de données local, contenant toutes les entrées locales qui se trouvent dans la corbeille, telles qu'elles ont été traitées par le client sans générer d'erreur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [\_\_trashbin_*LocalDataType*_complete\_\_.json](#__trashbin_localdatatype_complete__json)

## \_\_trashbin_*LocalDataType*_complete\_\_.json

Uniquement si la corbeille est activée. Un fichier par type de données local, contenant toutes les entrées locales qui se trouvent dans la corbeille, telles qu'elles sont sur le serveur.

Lorsque la file d'erreurs est vide, doit avoir le même contenu que [\_\_trashbin_*LocalDataType*.json](#__trashbin_localdatatypejson)
