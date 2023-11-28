---
title: Presentation
weight: 1
---

## What is Hermes

Hermes is a [Change Data Capture (CDC)](https://medium.com/event-driven-utopia/a-gentle-introduction-to-event-driven-change-data-capture-683297625f9b) tool from any source(s) to any target(s).

{{% notice warning %}}
**This project is still under developpement and is not ready for production use yet**
{{% /notice %}}

### Simplified process flow

Hermes-server will regularly poll data from data source(s), and generate a diff between the fresh dataset and the previous one stored in cache. Each difference will be converted into an Event message, and sent to a message bus (ie. [Kafka](https://kafka.apache.org/), [RabbitMQ](https://www.rabbitmq.com/)...).

The clients will receive and process each Event message to propagate data on their respective targets.

``` mermaid
flowchart LR
  subgraph Datasources
    direction LR
    RefOracle
    RefPostgreSQL
    RefLDAP
    RefEtc
  end
  subgraph Hermes-server
    direction LR
    hermes-server
  end
  subgraph External_dependencies["External dependencies"]
    direction LR
    MessageBus
  end
  subgraph Hermes-clients
    direction LR
    hermes-client-ldap
    hermes-client-aspypsrp-ad
    hermes-client-webservice
    hermes-client-etc["..."]
  end
  subgraph Targets
    direction LR
    LDAP
    ActiveDirectory
    webservice
    etc
  end
  RefOracle[(Oracle source)]-->|Data|hermes-server
  RefPostgreSQL[(PostgreSQL source)]-->|Data|hermes-server
  RefLDAP[(LDAP source)]-->|Data|hermes-server
  RefEtc[(...)]-->|Data|hermes-server
  hermes-server-->|Events|MessageBus((MessageBus))
  MessageBus-->|Events|hermes-client-ldap
  MessageBus-->|Events|hermes-client-aspypsrp-ad
  MessageBus-->|Events|hermes-client-webservice
  MessageBus-->|Events|hermes-client-etc
  hermes-client-ldap-->|Update|LDAP[(LDAP)]
  hermes-client-aspypsrp-ad-->|Update|ActiveDirectory[(Active Directory)]
  hermes-client-webservice-->|Update|webservice[(Web service <i>name</i>)]
  hermes-client-etc-->|Update|etc[("...")]


  classDef external fill:#fafafa,stroke-dasharray: 5 5
  class Datasources,External_dependencies,Targets external

  
```
