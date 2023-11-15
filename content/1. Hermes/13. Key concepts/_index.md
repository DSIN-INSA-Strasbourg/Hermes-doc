---
title: Key concepts
url: hermes/key-concepts
weight: 13
---

## Datasource

A source from which the [server](#server) will collect data. Can be anything that contains data : database, LDAP directory, web service, flat file...

## Datasource plugin

An Hermes plugin ran by [server](#server) in charge of collecting the data from a specific [datasource](#datasource) type and providing it to the [server](#server).

## Server

hermes-server application : will poll datasources at regular intervals and convert all changes between fresh data and previous one into events that will be sent on [message bus](#message-bus) by [message bus producer plugin](#message-bus-producer-plugin).

## Message bus

External service like [Apache Kafka](https://kafka.apache.org/) or [RabbitMQ](https://www.rabbitmq.com/) that will collect events from [server](#server) and provide them to the [clients](#client) in the same order that they had been emitted.

## Message bus producer plugin

An Hermes plugin ran by [server](#server) in charge of emitting events on a specific [message bus](#message-bus) type.

## Message bus consumer plugin

An Hermes plugin ran by [clients](#client) in charge of consuming events from a specific [message bus](#message-bus) type.

## Client

hermes-client application : will consume events from [message bus](#message-bus) by [message bus consumer plugin](#message-bus-consumer-plugin) and call appropriate methods implemented on [client plugin](#client-plugin) to propagate data changes on target.

### Trashbin

If configured to, the client won't immediately remove data, but store it in trashbin for a configured number of days. If the data is added again before this delay, the client will restore it from trashbin. Otherwise, once the trashbin retention limit will be reached, the data will be removed.

Depending on the chosen implementation on [client plugin](#client-plugin), it may allow to disable data, or to keep it as is for a longer time.

### Error queue

When an exception is raised during an event processing on [client plugin](#client-plugin), the event is stored on an error queue. All new events about first's data objects won't be processed but stored on error queue. The processing of events in error queue is retried periodically.

### Auto remediation

Sometimes, an event may be stored on [error queue](#error-queue) due to a data problem (ie. a group name with a trailing dot will raise an error on Active Directory). If the trailing dot is removed from the group name on [datasource](#datasource), the `modified` event will be stored on [error queue](#error-queue), and won't be processed, and that won't happen without manually editing [client](#client)'s cache file.

The autoremediation solves this type of problems by merging data of `added` and `modified` events of a same object in [error queue](#error-queue).
It is not enabled by default, as it break the "*all events are processed in the order they were generated*" rule.

## Client plugin

An Hermes plugin ran by [client](#client) in charge of implementing simple event processing methods to propagate data changes to a specific target type.

## Attribute plugin

An Hermes plugin ran by [server](#server) or [client](#client) that will be offered as a new [Jinja filter](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters), allowing data transformation.

## Initsync

A [client](#client) is unable to start processing new events safely without disposing of the previous complete data set. So the [server](#server) is able to send a specific event sequences called `initsync` that will contains the [server's datamodel](#servers-datamodel) and the whole data set. The already initialized client will silently ignore this sequence, but the uninitialized will process it to initialize by adding all entries provided by initsync, and will then process subsequent events.

## Datamodel

As they're little differences between them, please see [server's datamodel](#servers-datamodel) and [client's datamodel](#clients-datamodel).

### Data type

Also named "*object type*". A type of data with its [attributes mapping](#attributes-mapping) to be handled by Hermes.

#### Primary key

The [data types](#data-type)'s attribute that is used to distinguish an entry from the others. It must obviously be unique.

### Server's datamodel

Configuration of [data types](#data-type) that [server](#server), with their [attributes mapping](#attributes-mapping). The remote attribute name is the attribute name used on [datasource](#datasource).

The server's datamodel is built by specifying the following items :

- Each [data types](#data-type) with :
  - its [primary key](#primary-key)
  - its [integrity-constraints](#integrity-constraints)
  - its [merge conflict policy](#merge-conflict-policy)
  - each of its [datasources](#datasource) name and operations with :
    - its [attributes mapping](#attributes-mapping)
    - its special attributes list : [local attributes](#local-attributes), [secrets attributes](#secrets-attributes) and [cache-only attributes](#cache-only-attributes)
    - its [merge-constraints](#merge-constraints)

#### Merge conflict policy

Define the behavior if a same attribute has different value on different [datasources](#datasource).

#### Merge constraints

Allow to declare some constraints to ensure data consistency during data merge, when [server](#server) is polling data from multiple [datasources](#datasource).

#### Integrity constraints

Allow to declare some constraints between several [data types](#data-type) to ensure data consistency.

##### Cache only attributes

[Datamodel](#servers-datamodel)'s attributes that will only be stored in cache, but won't be sent in events, nor used to diff with cache.

##### Secrets attributes

[Datamodel](#servers-datamodel)'s attributes that will contain sensitive data, like passwords and must never be stored in cache nor printed in logs.  They'll be sent to clients unless they're defined as [local attributes](#local-attributes) too.

{{% notice style="note" %}}
As those attributes are not cached, they'll be seen as added at EACH [server](#server)'s polling.
{{% /notice %}}

##### Local attributes

[Datamodel](#servers-datamodel)'s attributes that won't be sent in events, cached or used to diff with cache.

### Client's datamodel

Configuration of [data types](#data-type) that [client](#client) must handle, with their [attributes mapping](#attributes-mapping). The remote attribute name is the attribute name used on the [server's datamodel](#servers-datamodel).

{{% notice style="info" %}}
If you're asking yourself why this mapping is necessary, here's why :

1. it allow local data transformation via [Jinja filter](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters) and [Attribute plugin](#attribute-plugin) on [client](#client).
2. it allow re-using (and sharing) [client plugins](#client-plugin) without requiring any change to your [server's datamodel](#servers-datamodel) nor plugin code, but simply by changing [client](#client)'s configuration file.

{{% /notice %}}

The client's datamodel is built by specifying the following items :

- Each [data types](#data-type) with :
  - its corresponding remote [data type](#data-type) name called `hermesType`
  - its [attributes mapping](#attributes-mapping)

### Attributes mapping

Also named "*attrsmapping*". Mapping (key/value) that link the internal attribute name (as key) with the remote one (as value). The remote may possibly contain a one liner Jinja template to process data transformation via [Jinja filter](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters) and [Attribute plugin](#attribute-plugin).
