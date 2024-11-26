---
title: Foreign keys
weight: 3
---

Sometimes, objects are linked together by foreign keys. When an error occurs on an object whose primary key refers to that of one or more other "parent" objects, it may be desirable to interrupt the processing of all or part of the events of these parent objects until this first event has been correctly processed. This can be done by adding the events of the parent objects to the error queue instead of trying to process them.

The first thing to do is to declare the foreign keys through [hermes-server.datamodel.data-type-name.foreignkeys](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.foreignkeys) in hermes-server configuration. The server will do nothing with these foreign keys except propagate them to the clients.

Then, it is necessary to establish which policy to apply to the clients through [hermes-client.foreignkeys_policy](/setup/configuration/hermes-client/#hermes-client.foreignkeys_policy) in each hermes-client configuration. There are three:

- `disabled`: No event, policy is disabled. Probably not relevant in most cases, but could perhaps be useful to someone one day.
- `on_remove_event`: Only on *removed* events. Should be enough in most cases.
- `on_every_event`: On every event types (*added*, *modified*, *removed*). To ensure perfect consistency no matter what.
