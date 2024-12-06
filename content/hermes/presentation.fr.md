---
title: Présentation
weight: 1
---

## Qu'est-ce qu'Hermes ?

Hermes est un outil de [capture des changements de données (CDC)](https://medium.com/event-driven-utopia/a-gentle-introduction-to-event-driven-change-data-capture-683297625f9b) depuis n'importe quelles sources vers n'importe quelles cibles.

### Flux de fonctionnement simplifié

Le serveur Hermes interrogera régulièrement les sources de données et générera un différentiel entre le nouvel ensemble de données et le précédent issu du cache. Chaque différence sera convertie en un message d'événement et envoyée à un bus de messages (*e.g.* [Kafka](https://kafka.apache.org/), [RabbitMQ](https://www.rabbitmq.com/)...).

Les clients recevront et traiteront chaque message d'événement pour propager les données vers leurs cibles respectives.

``` mermaid
flowchart LR
  subgraph Datasources["Sources de données"]
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
  subgraph External_dependencies["Dépendences externes"]
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
  subgraph Targets["Cibles"]
    direction LR
    LDAP
    ActiveDirectory
    webservice
    etc
  end
  RefOracle[(Oracle)]-->|Données|hermes-server
  RefPostgreSQL[(PostgreSQL)]-->|Données|hermes-server
  RefLDAP[(LDAP)]-->|Données|hermes-server
  RefEtc[(...)]-->|Données|hermes-server
  hermes-server-->|Événements|MessageBus(("Bus de message"))
  MessageBus-->|Événements|hermes-client-ldap
  MessageBus-->|Événements|hermes-client-aspypsrp-ad
  MessageBus-->|Événements|hermes-client-webservice
  MessageBus-->|Événements|hermes-client-etc
  hermes-client-ldap-->|Met à jour|LDAP[(LDAP)]
  hermes-client-aspypsrp-ad-->|Met à jour|ActiveDirectory[(Active Directory)]
  hermes-client-webservice-->|Met à jour|webservice[(Web service <i>nom</i>)]
  hermes-client-etc-->|Met à jour|etc[("...")]

  classDef external fill:#fafafa,stroke-dasharray: 5 5
  class Datasources,External_dependencies,Targets external
```
