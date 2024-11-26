---
title: Client datamodel update
weight: 2
---

A data model is not fixed in time, it can evolve and therefore be updated, whether from the server or on one or more clients.

Each time the datamodel is modified on a client, the client will generate appropriate local events to reflect the data changes on targets.  
It may notify about datamodel warnings if some remote data type or attributes are set in its datamodel, but doesn't exist in current dataschema received from hermes-server.

## Add an attribute to an existing data type

1. 👱 Add attribute to [clients datamodel](/hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of "modified" local events from the complete cache

## Remove an attribute from a data type

1. 👱 Remove attribute from [clients datamodel](/hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of consecutive "modified" local events

## Modify the value of an attribute (by changing its Jinja filter, or its remote attribute from the data source)

1. 👱 Modify attribute in [clients datamodel](/hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of consecutive "modified" local events

## Add a new data type

### If its *hermesType* already exists in dataschema

1. 👱 Add data type to [clients datamodel](/hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of "added" local events from the complete cache

### If its *hermesType* doesn't exists in dataschema yet

1. 👱 Add data type to [clients datamodel](/hermes/key-concepts/#client-datamodel) so that they can process it when it will be added to the [server datamodel](/hermes/key-concepts/#server-datamodel), reload clients: ⚠️ new datamodel warning "*remote types don't exist in current Dataschema*"
2. 👱 Add data type to [server datamodel](/hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](/hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "added" events for each entry of added data type
3. 💻 Processing of [dataschema event](/hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema. ✅ No more datamodel warning. Processing incoming "added" events

## Remove an existing data type

1. 👱 Remove data type from [clients datamodel](/hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of consecutive "removed" local events
    - 💻 Purging local cache files of removed data type
