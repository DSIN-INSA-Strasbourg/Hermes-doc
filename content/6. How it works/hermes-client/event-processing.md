---
title: Event processing
url: how-it-works/hermes-client/event-processing
weight: 2
---

As the datamodel on server differs than that on client, the clients must convert *remote* events received on message bus to *local* events. If the resulting local event is empty (the data type or the attributes changed in remote event are not set on client datamodel), the event is ignored.

On client datamodel update, the client may generate local events that have no corresponding remote event, *ie* to update an attribute value computed with a Jinja template that just had been updated.

``` mermaid
flowchart TB
  subgraph Hermes-client
    direction TB
    datamodelUpdate[["a datamodel update"]]
    remoteevent["Remote event"]
    localevent["Local event"]
    eventHandler(["Client plugin event handler"])
  end
  datamodelUpdate-->|generate|localevent
  MessageBus-->|produce|remoteevent
  remoteevent-->|convert to|localevent
  localevent-->|pass to appropriate|eventHandler
  eventHandler-->|process|Target

  classDef external fill:#fafafa,stroke-dasharray: 5 5
  class MessageBus,Target external
```
