---
title: Server datamodel update
weight: 1
---

A data model is not fixed in time, it can evolve and therefore be updated, whether from the server or on one or more clients.

Each time the datamodel is modified on the server, its new version is propagated to the clients with its "public" data: each data type is included, with its primary key, the list of its attributes, and the list of its secret attributes. Then some consecutive events are emitted.

## Add an attribute to an existing data type

1. 👱 Add attribute to [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "modified" events for the concerned entries, with the added attribute and its value
    - 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema. Processing incoming "modified" events: as the attribute is not declared yet in their datamodel, its value is ignored but stored in the complete cache
2. 👱 Add attribute to [clients datamodel](../../hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of "modified" local events from the complete cache

or

1. 👱 Add attribute to [clients datamodel](../../hermes/key-concepts/#client-datamodel) so that they can process it when it will be added to the [server datamodel](../../hermes/key-concepts/#server-datamodel), reload clients: ⚠️ datamodel warning "*remote attributes don't exist in current Dataschema*"
2. 👱 Add attribute to [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "modified" events for the concerned entries, with the added attribute and its value
3. 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema. ✅ No more datamodel warning. Processing incoming "modified" events

## Remove an attribute from a data type

1. 👱 Remove attribute from [clients datamodel](../../hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of consecutive "modified" local events
2. 👱 Remove attribute from [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "modified" events for the concerned entries, with the removed attribute. They'll be ignored by clients
3. 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema

or

1. 👱 Remove attribute from [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "modified" events for the concerned entries, with the removed attribute
2. 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema. ⚠️ new datamodel warning "*remote attributes don't exist in current Dataschema*". Processing incoming "modified" events
3. 👱 Remove attribute from [clients datamodel](../../hermes/key-concepts/#client-datamodel), reload clients: ✅ No more datamodel warning

## Modify the value of an attribute (by changing its Jinja filter, or its remote attribute from the data source)

1. 👱 Modify attribute in [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of "modified" events for the concerned entries, with the modified attribute new values
2. 💻 Processing incoming "modified" events

## Add an existing attribute of a data type to *[secrets_attrs](../../setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)*

1. 👱 Modify *[secrets_attrs](../../setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)* in [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Purging attribute from server cache
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "modified" events for the concerned entries, with the "added" attribute and its values
2. 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema, purging attribute from their cache
    - 💻 Processing incoming "modified" events

## Remove an existing attribute of a data type from *[secrets_attrs](../../setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)*

1. 👱 Modify *[secrets_attrs](../../setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)* in [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "modified" events for the concerned entries, with the "added" attribute and its values
2. 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema
    - 💻 Processing incoming "modified" events

## Add a new data type

1. 👱 Add data type to [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "added" events for each entry of added data type
    - 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema. Processing incoming "added" events: as the data type is not declared yet in their datamodel, its entries are ignored but stored in the complete cache
2. 👱 Add data type to [clients datamodel](../../hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of "added" local events from the complete cache

or

1. 👱 Add data type to [clients datamodel](../../hermes/key-concepts/#client-datamodel) so that they can process it when it will be added to the [server datamodel](../../hermes/key-concepts/#server-datamodel), reload clients: ⚠️ new datamodel warning "*remote types don't exist in current Dataschema*"
2. 👱 Add data type to [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Emission of "added" events for each entry of added data type
3. 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema. ✅ No more datamodel warning. Processing incoming "added" events

## Remove an existing data type

1. 👱 Remove data type from [clients datamodel](../../hermes/key-concepts/#client-datamodel), reload clients
    - 💻 Local datamodel update processing by clients: generation and processing of consecutive "removed" local events
    - 💻 Purging local cache files of removed data type
2. 👱 Remove data type from [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of "removed" events for each entry of removed data type
    - 💻 Purging cache files of removed data type
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
3. 💻 Processing incoming "removed" events by clients: all are ignored
    - 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema
    - 💻 Purging remote cache files of removed data type

or

1. 👱 Remove data type from [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Emission of "removed" events for each entry of removed data type
    - 💻 Purging cache files of removed data type
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
2. 💻 Processing incoming "removed" events by clients
    - 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema. ⚠️ new datamodel warning "*remote types don't exist in current Dataschema*"
    - 💻 Purging remote cache files of removed data type
3. 👱 Remove data type from [clients datamodel](../../hermes/key-concepts/#client-datamodel), reload clients: ✅ No more datamodel warning
    - 💻 Purging local cache files of removed data type

## Change the primary key attribute of a data type

{{% notice style="warning" title="DANGER - Here be dragons" %}}
This is the riskiest datamodel update, as there may be links between data types, using the primary key as a foreign key.  
This means that you'll need to update every data type at once, without missing anything.

The new primary key values must exist in every entry on every client beforehand.

**You should really consider doing this update on a test environment before doing it in production**, because if something fails, your clients could be permanently broken.
{{% /notice %}}

### Prerequisites

The attribute(s) to use as new primary key must already exist in your [server datamodel](../../hermes/key-concepts/#server-datamodel), and in all [clients datamodel](../../hermes/key-concepts/#client-datamodel) as a **raw value** (without Jinja template), and their value must already have been propagated and exists in clients cache.

{{% notice style="tip" title="Trashbin retention may delay the Datamodel update" %}}
The new primary key **MUST** exist in every entry of its data type before updating the datamodel. If trashbin is enabled on some of your clients, the new primary key attribute might be missing from trashed entries.

The safest way to handle this is to add the attribute to your [server datamodel](../../hermes/key-concepts/#server-datamodel) and delay the primary key change at least to one day + as many days as the highest [trashbin_retention](../../setup/configuration/hermes-client/#hermes-client.trashbin_retention) value of all your clients.
{{% /notice %}}

### Updating

1. 👱 Update all data types in [server datamodel](../../hermes/key-concepts/#server-datamodel), reload server
    - 💻 Updating changed primary keys in cache files on the server
    - 💻 Emission of a [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by the server
    - 💻 Processing of [dataschema event](../../hermes/how-it-works/hermes-server/events-emitted/) by clients: updating their schema, updating changed primary keys in error queue and cache files
