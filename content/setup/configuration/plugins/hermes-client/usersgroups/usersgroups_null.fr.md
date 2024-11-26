---
title: "null"
---

## Description

Ce client traite les événements de type Users, Groups and UserPasswords, mais ne fait rien d'autre que de générer des logs.

## Configuration

Rien à configurer pour le plugin.

```yaml
hermes-client-usersgroups_null:
```

## Datamodel

Les types de données suivants peuvent être configurés, sans contrainte particulière puisque rien ne sera traité.

- Users
- UserPasswords
- Groups
- GroupsMembers

```yaml
  datamodel:
    Users:
      hermesType: your_server_Users_type_name
      attrsmapping:
        attr1_client:  attr1_server
        # ...

    UserPasswords:
      hermesType: your_server_UserPasswords_type_name
      attrsmapping:
        attr1_client:  attr1_server
        # ...

    Groups:
      hermesType: your_server_Groups_type_name
      attrsmapping:
        attr1_client:  attr1_server
        # ...

    GroupsMembers:
      hermesType: your_server_GroupsMembers_type_name
      attrsmapping:
        attr1_client:  attr1_server
        # ...
```
