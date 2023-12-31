---
title: ldap
---

## Description

This plugin allows the use of an LDAP server as datasource.

## Configuration

Connection settings are required in plugin configuration.

```yaml
hermes:
  plugins:
    datasources:
      # Source name. Use whatever you want. Will be used in datamodel
      your_source_name:
        type: ldap
        settings:
          # MANDATORY: LDAP server URI
          uri: ldaps://ldap.example.com:636
          # MANDATORY: LDAP server credentials to use
          binddn: cn=account,dc=example,dc=com
          bindpassword: s3cReT_p4s5w0rD
          # MANDATORY: LDAP base DN
          basedn: dc=example,dc=com

          ssl: # Facultative
            # Path to PEM file with CA certs
            cafile: /path/to/INTERNAL-CA-chain.crt # Facultative
            # Path to file with PEM encoded cert for client cert authentication,
            # requires keyfile
            certfile: /path/to/client.crt # Facultative
            # Path to file with PEM encoded key for client cert authentication,
            # requires certfile
            keyfile: /path/to/client.pem # Facultative

          # Facultative. Default: false.
          # Since the client is not aware of the LDAP schema, it cannot know whether
          # an attribute is single-valued or multi-valued. By default, it will
          # return a single value in its base type, as if it were a single-valued
          # attribute, and multiple values in a list.
          # If this setting is enabled, all values will always be returned in a list.
          always_return_values_in_list: true
```

## Usage

Usage differs according to specified operation type

### fetch

Fetch entries from LDAP server.

```yaml
hermes-server:
  datamodel:
    oneDataType:
      sources:
        your_source_name: # 'your_source_name' was set in plugin settings
          fetch:
            type: fetch
            vars:
              # Facultative: the basedn to use for 'fetch' operation.
              # If unset, setting basedn will be used
              base: "ou=exampleOU,dc=example,dc=com"
              # Facultative: the operation scope for 'fetch' operation
              # Valid values are:
              # - base: to search the "base" object itself
              # - one, onelevel: to search the "base" object’s immediate children
              # - sub, subtree: to search the "base" object and all its descendants
              # If unset, "subtree" will be used
              scope: subtree
              # Facultative: the LDAP filter to use for 'fetch' operation
              # If unset, "(objectClass=*)" will be used
              filter: "(objectClass=*)"
              # Facultative: the attributes to fetch, as a list of strings
              # If unset, all the attributes of each entry are returned
              attrlist: "{{ REMOTE_ATTRIBUTES }}"
```

### add

Add entries to LDAP server.

```yaml
hermes-server:
  datamodel:
    oneDataType:
      sources:
        your_source_name: # 'your_source_name' was set in plugin settings
          fetch:
            type: add
            vars:
              # Facultative: a list of entries to add.
              # If unset, an empty list will be used (and nothing will be added)
              addlist:
                  # MANDATORY: the DN of the entry. If not specified, the entry will
                  # be silently ignored
                - dn: uid=newentry1,ou=exampleOU,dc=example,dc=com
                  # Facultative: the attributes to add to the entry
                  add:
                    # Create attribute if it doesn't exist, and add "value" to it
                    "attrnameToAdd": "value",
                    # Create attribute if it doesn't exist, and add "value1" and
                    # "value2" to it
                    "attrnameToAddList": ["value1", "value2"],
                - dn: uid=newentry2,ou=exampleOU,dc=example,dc=com
                  # ...
```

### delete

Delete entries from LDAP server.

```yaml
hermes-server:
  datamodel:
    oneDataType:
      sources:
        your_source_name: # 'your_source_name' was set in plugin settings
          fetch:
            type: delete
            vars:
              # Facultative: a list of entries to delete.
              # If unset, an empty list will be used (and nothing will be deleted)
              dellist:
                  # MANDATORY: the DN of the entry. If not specified, the entry will
                  # be silently ignored
                - dn: uid=entryToDelete1,ou=exampleOU,dc=example,dc=com
                - dn: uid=entryToDelete2,ou=exampleOU,dc=example,dc=com
                  # ...
```

### modify

Modify entries on LDAP server.

```yaml
hermes-server:
  datamodel:
    oneDataType:
      sources:
        your_source_name: # 'your_source_name' was set in plugin settings
          fetch:
            type: modify
            vars:
              # Facultative: a list of entries to modify.
              # If unset, an empty list will be used (and nothing will be modified)
              modlist:
                  # MANDATORY: the DN of the entry. If not specified, the entry will
                  # be silently ignored
                - dn: uid=entryToModify1,ou=exampleOU,dc=example,dc=com

                  # Facultative: the attributes to add to the entry
                  add:
                    # Create attribute if it doesn't exist, and add "value" to it
                    attrnameToAdd: value
                    # Create attribute if it doesn't exist, and add "value1" and
                    # "value2" to it
                    attrnameToAddList: [value1, value2]

                  # Facultative: the attributes to modify in the entry
                  modify:
                    # Create attribute if it doesn't exist, and replace all its
                    # value by "value"
                    attrnameToModify: newvalue
                    # Create attribute if it doesn't exist, and replace all its
                    # value by "newvalue1" and "newvalue2"
                    attrnameToModifyList: [newvalue1, newvalue2]

                  # Facultative: the attributes to delete from the entry
                  delete:
                    # Delete specified attribute and all of its values
                    attrnameToDelete: null
                    # Delete "value" from specified attribute. Raise an error if
                    # value is missing
                    attrnameToDeleteValue: value
                    # Delete "value1" and "value2" from specified attribute. Raise
                    # an error if a value is missing
                    attrnameToDeleteValueList: [value1, value2]

                - dn: uid=entryToModify2,ou=exampleOU,dc=example,dc=com
                  # ...
```
