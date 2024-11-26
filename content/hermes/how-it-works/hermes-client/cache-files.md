---
title: Cache files
weight: 5
---

## _hermes-client-*name*.json

Contains state of the client:

- ```py
  queueErrors: dict[str, str]
  ```

  Dictionary containing all error messages of objects in error queue, to be able to notify of any changes.

- ```py
  datamodelWarnings: dict[str, dict[str, dict[str, Any]]]
  ```

  Dictionary containing current datamodel warnings, for notifications.

- ```py
  exception: str | None
  ```

  String containing latest exception trace.

- ```py
  initstartoffset: Any | None
  ```

  Contains the offset of the first message of initSync sequence on message bus.

- ```py
  initstopoffset: Any | None
  ```

  Contains the offset of the last message of initSync sequence on message bus.

- ```py
  nextoffset: Any | None
  ```

  Contains the offset of the next message to process on message bus.

## \_hermesconfig.json

Cache of previous config, used to be able to build the previous datamodel and to render the Jinja templates with Attribute plugins.

## \_dataschema.json

Cache of latest Dataschema, received from hermes-server.

## \_errorqueue.json

Cache of error queue.

## *RemoteDataType*.json

One file per remote data type, containing all remote entries, as they had been successfully processed.

When error queue is empty, must have the same content than [*RemoteDataType*_complete\_\_.json](#remotedatatype_complete__json)

## *RemoteDataType*_complete\_\_.json

One file per remote data type, containing all remote entries, as they should be without error.

When error queue is empty, must have the same content than [*RemoteDataType*.json](#remotedatatypejson)

## trashbin_*RemoteDataType*.json

Only if trashbin is enabled. One file per remote data type, containing all remote entries that are in trashbin, as they had been successfully processed.

When error queue is empty, must have the same content than [trashbin_*RemoteDataType*_complete\_\_.json](#trashbin_remotedatatype_complete__json)

## trashbin_*RemoteDataType*_complete\_\_.json

Only if trashbin is enabled. One file per remote data type, containing all remote entries that are in trashbin, as they should be without error.

When error queue is empty, must have the same content than [trashbin_*RemoteDataType*.json](#trashbin_remotedatatypejson)

## \_\_*LocalDataType*.json

One file per local data type, containing all local entries, as they had been successfully processed.

When error queue is empty, must have the same content than [\_\_*LocalDataType*_complete\_\_.json](#__localdatatype_complete__json)

## \_\_*LocalDataType*_complete\_\_.json

One file per local data type, containing all local entries, as they should be without error.

When error queue is empty, must have the same content than [\_\_*LocalDataType*.json](#__localdatatypejson)

## \_\_trashbin_*LocalDataType*.json

Only if trashbin is enabled. One file per local data type, containing all local entries that are in trashbin, as they had been successfully processed.

When error queue is empty, must have the same content than [\_\_trashbin_*LocalDataType*_complete\_\_.json](#__trashbin_localdatatype_complete__json)

## \_\_trashbin_*LocalDataType*_complete\_\_.json

Only if trashbin is enabled. One file per local data type, containing all local entries that are in trashbin, as they should be without error.

When error queue is empty, must have the same content than [\_\_trashbin_*LocalDataType*.json](#__trashbin_localdatatypejson)
