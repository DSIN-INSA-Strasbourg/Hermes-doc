---
title: Cache files
weight: 6
---

## _hermes-server.json

Contains state of the server :

- ```py
  lastUpdate: datetime.datetime | None
  ```

  Datetime of latest update.

- ```py
  errors: dict[str, dict[str, dict[str, Any]]]
  ```

  Dictionnary containing current errors, for notifications.

- ```py
  exception: str | None
  ```

  String containg latest exception trace.

## _dataschema.json

Contains the Dataschema, built upon the Datamodel. This cache file permit to server to process to the step `2.` from [Workflow](../workflow/).

## *DataType*.json

There's one file per data type declared in Datamodel, containing the data cache of this data type, as a list of dict. Each dict from the list is an entry.
