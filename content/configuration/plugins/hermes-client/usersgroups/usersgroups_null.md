---
title: "null"
url: configuration/plugins/hermes-client/usergroups/null
---

## Description

This client will handle Users, Groups and UserPasswords events, but does nothing but logging.

## Configuration

Nothing to configure for the plugin.

```yaml
hermes-client-usersgroups_null:
```

## Datamodel

The following data types may be set up, without any specific constraint as nothing will be processed.

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
