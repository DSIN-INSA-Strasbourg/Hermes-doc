---
title: Features
weight: 2
---

- Doesn't require any change to sources data model (ie. no need to add a `last_updated` column)
- Multi-source, with ability to [merge](../hermes/how-it-works/hermes-server/data-merging/) or [aggregate](../hermes/how-it-works/hermes-server/data-aggregation/) data, and optionnally set merge/aggregation constraints
- Able to handle several data types, with link (*foreign keys*) between them, and to enforce [integrity constraints](../hermes/how-it-works/hermes-server/integrity-constraints/)
- Able to transform data with [Jinja filters](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters)
- Clean error handling, to avoid synchronization problems, and an optional mechanism of error [auto remediation](../hermes/how-it-works/hermes-client/auto-remediation/)
- Offer a [trashbin](../hermes/key-concepts/#trashbin) on clients for removed data
- Insensitive to unavailability and errors on each link (source, message bus, target)
- Evolutive by design. All following items are implemented as [plugins](development/plugins/) ([list of existing plugins](setup/configuration/plugins/)) :
  - Datasources
  - Attributes filters (data filters)
  - Clients (targets)
  - Messagebus
