---
title: Features
weight: 2
---

- Doesn't require any change to the source data model(s) (ie. no need to add a `last_updated` column)
- Multi-source, with ability to [merge](../hermes/how-it-works/hermes-server/data-merging/) or [aggregate](../hermes/how-it-works/hermes-server/data-aggregation/) data, and optionally set merge/aggregation constraints
- Able to handle several data types, with link (*foreign keys*) between them, and to enforce [integrity constraints](../hermes/how-it-works/hermes-server/integrity-constraints/)
- Able to transform data with [Jinja filters](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters) in configuration files : no need to edit some Python code
- Clean error handling, to avoid synchronization problems, and an optional mechanism of error [auto remediation](../hermes/how-it-works/hermes-client/auto-remediation/)
- Offer a [trashbin](../hermes/key-concepts/#trashbin) on clients for removed data
- Insensitive to unavailability and errors on each link (source, message bus, target)
- Easy to extend by design. All following items are implemented as [plugins](development/plugins/) ([list of existing plugins](setup/configuration/plugins/)) :
  - Datasources
  - Attribute filters (data filters)
  - Clients (targets)
  - Messagebus
- Changes to the datamodel are easy and safe to integrate and propagate, whether on the [server](../maintenance/server-datamodel-update/) or on the [clients](../maintenance/client-datamodel-update/)
