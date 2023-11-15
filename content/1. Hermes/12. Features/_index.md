---
title: Features
url: hermes/features
weight: 12
---

- Doesn't require any change to sources data model (ie. no need to add `last_updated` column)
- [Multi-source](#multi-source), with ability to set [merge](#merging-data)/[aggregation](#aggregating-data) constraints
- Able to handle several data types, with link (*foreign keys*) between them, and to enforce [integrity constraints](#integrity-constraints)
- Able to transform data with [Jinja filters](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters)
- Clean error handling, to avoid synchronization problems, and an optional mechanism of error remediation
- Offer a trashbin on clients for removed data
- Insensitive to unavailability and errors on each link (source, message bus, target)
- Evolutive by design. All following items are implemented as plugins :
  - Datasources
  - Attributes filters (data filters)
  - Clients (targets)
  - Messagebus

## Multi-source

Hermes-server is able to poll multiple sources and merge or aggregate their data as if it was providing of the same source. It is even possible to set merge/aggregation constraints to ensure data consistency.

### Merging data

Let's work with a university data set, where Hermes should manage user accounts and employees and students are stored in different datasources. Hermes will be able to merge the two datasources in one virtual `Users`, but must ensure that primary keys are differents.

Here we got two distinct data sources for a same data type.

```mermaid
classDiagram
    direction BT
    Users <|-- Users_employee
    Users <|-- Users_students
    class Users{
      user_id
      login
      mail
      merge_constraints() s.user_id mustNotExist in e.user_id
    }
    class Users_students{
      s.user_id
      s.login
      s.mail
    }
    class Users_employee{
      e.user_id
      e.login
      e.mail
    }
```

### Aggregating data

Another use case, where Hermes should manage user accounts and main data and wifi profile's name are stored in different datasources. Hermes will be able to aggregate the two datasources in one virtual `Users`, but must ensure that primary keys of second exists in first.

Here we got two distinct data sources for a same entry.

```mermaid
classDiagram
    direction BT
    Users <|-- Users_main
    Users <|-- Users_auxiliary
    class Users{
      user_id
      login
      mail
      wifi_profile
      merge_constraints() a.user_id mustAlreadyExist in m.user_id
    }
    class Users_auxiliary{
      a.user_id
      a.wifi_profile
    }
    class Users_main{
      m.user_id
      m.login
      m.mail
    }
```

## Integrity constraints

Hermes-server is able to handle several data types, with link (*foreign keys*) between them, and to enforce integrity constraints.

Let's use a typical Users / Groups / GroupsMember use case to illustrate this.

```mermaid
classDiagram
    direction BT
    GroupsMembers <-- Users
    GroupsMembers <-- Groups
    class Users{
      user_id
      ...
    }
    class Groups{
      group_id
      ...
    }
    class GroupsMembers{
      user_id
      group_id
      integrity() _SELF.user_id in Users_pkeys and _SELF.group_id in Groups_pkeys
    }
```
