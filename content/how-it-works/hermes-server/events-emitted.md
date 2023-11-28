---
title: Events emitted
weight: 2
---

### Event categories

An event always belong to one of those categories :

- `base` : standard event, can be of type :
  - `dataschema` : propagate the new dataschema to clients, after a server datamodel update
  - `added` : a new entry has been added to specified data type, with specified attributes and values
  - `removed` : entry of specified pkey has been removed from specified data type
  - `modified` : entry of specified pkey has been modified. Contains only added, modified and removed attributes with their new values

- `initsync` : indicate that the event is a part of an initsync sequence, can be of type :
  - `init-start` : beginning of an initsync sequence, also contains the current dataschema
  - `added` : a new entry has been added to specified data type, with specified attributes and values. As the server sends the contents of its cache to initialize clients, entries can only be added
  - `init-stop` : end of an initsync sequence
