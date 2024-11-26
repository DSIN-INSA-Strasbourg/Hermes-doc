---
title: Integrity constraints
weight: 2
---

Hermes-server can handle several data types, with link (*[foreign keys](/hermes/how-it-works/hermes-client/foreign-keys/)*) between them, and to enforce integrity constraints.

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

In this scenario, entries in `GroupsMembers` that have a `user_id` that doesn't exist in `Users`, or a `group_id` that doesn't exist in `Groups` will be silently ignored.

For more details, please see [integrity_constraints](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints) in hermes-server configuration.
