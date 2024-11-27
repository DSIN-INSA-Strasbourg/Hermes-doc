---
title: Plugins
weight: 1
ordersectionsby: title
---

Whatever its type, a plugin is always a folder named '*plugin_name*' containing at least the following 4 files:

## Plugin source code

Hermes will try to import the ***plugin_name*.py** file. It is possible to split the plugin code into several files and folders, but the plugin will always be imported from this file.

For details about plugin API, please consult the following sections:

- [Attributes](/development/plugins/attributes/)
- [Clients](/development/plugins/clients/)
- [Datasources](/development/plugins/datasources/)
- [Messagebus consumers](/development/plugins/messagebus_consumers/)
- [Messagebus producers](/development/plugins/messagebus_producers/)

{{% notice tip %}}
Some helpers modules are available in `helpers`:

- `helpers.command`: to run local commands on client's host
- `helpers.randompassword`: to generate random passwords with specific constraints
{{% /notice %}}

## Plugin configuration schema

Depending on the plugin type, the configuration schema file slightly differs.

### Plugin configuration schema for clients plugins

Hermes will try to validate the plugin settings with a [Cerberus validation schema](https://docs.python-cerberus.org/schemas.html) specified in a YAML file: **config-schema-client-*plugin_name*.yml**.

The clients plugins validation file must be empty or contains only one top level key that must be the plugin name prefixed by `hermes-client-`.

Example for plugin name `usersgroups_flatfiles_emails_of_groups`:

```yaml { title="config-schema-client-usersgroups_flatfiles_emails_of_groups.yml" }
# https://docs.python-cerberus.org/validation-rules.html

hermes-client-usersgroups_flatfiles_emails_of_groups:
  type: dict
  required: true
  empty: false
  schema:
    destDir:
      type: string
      required: true
      empty: false
    onlyTheseGroups:
      type: list
      required: true
      nullable: false
      default: []
      schema:
        type: string
```

### Plugin configuration schema for other plugin types

Hermes will try to validate the plugin settings with a [Cerberus validation schema](https://docs.python-cerberus.org/schemas.html) specified in a YAML file: **config-schema-plugin-*plugin_name*.yml**.

Even if the plugin doesn't require any configuration, it still requires an empty validation file.

Example for plugin name `ldapPasswordHash`:

```yaml { title="config-schema-plugin-ldapPasswordHash.yml" }
# https://docs.python-cerberus.org/validation-rules.html

default_hash_types:
  type: list
  required: false
  nullable: false
  empty: true
  default: []
  schema:
    type: string
    allowed:
      - MD5
      - SHA
      - SMD5
      - SSHA
      - SSHA256
      - SSHA512

```

## Plugin README.md

The documentation should be written in **README.md** and should contains the following sections:

```md { title="README.md" }
# `plugin_name` attribute plugin

## Description

## Configuration

## Usage
Only for `attributes` and `datasources` plugins.

## Datamodel
Only for `clients` plugins.
```

## Plugin requirements.txt

Even if the plugin has no Python requirements, please create a pip `requirements.txt` file starting with a comment containing the plugin path and ending with an empty line.

Example:

```txt { title="requirements.txt" }
# plugins/attributes/ldapPasswordHash
passlib==1.7.4
 
```
