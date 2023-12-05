---
title: postgresql
---

## Description

This plugin allows using a PostgreSQL database as datasource.

## Configuration

Connection settings are required in plugin configuration.

```yaml
hermes:
  plugins:
    datasources:
      # Source name. Use whatever you want. Will be used in datamodel
      your_source_name:
        type: postgresql
        settings:
          # MANDATORY: the database server DNS name or IP address
          server: dummy.example.com
          # MANDATORY: the database connection port
          port: 1234
          # MANDATORY: the database name
          dbname: DUMMY
          # MANDATORY: the database credentials to use
          login: HERMES_DUMMY
          password: "DuMmY_p4s5w0rD"
```

## Usage

Specify a query. If you'd like to provide values from cache, you should provide them in a `vars` dict, and refer to them by specifying the var key name encased in `%()s` in the query: this will automatically sanitize the query. See example below.

The example `vars` names are prefixed with `sanitized_` only for clarity, it's not a requirement.

```yaml
hermes-server:
  datamodel:
    oneDataType:
      sources:
        your_source_name: # 'your_source_name' was set in plugin settings
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM A_POSTGRESQL_TABLE

          commit_one:
            type: modify
            query: >-
              UPDATE A_POSTGRESQL_TABLE
              SET
                valueToSet = %(sanitized_valueToSet)s
              WHERE pkey = %(sanitized_pkey)s

            vars:
              sanitized_pkey: "{{ ITEM_FETCHED_VALUES.pkey }}"
              sanitized_valueToSet: "{{ ITEM_FETCHED_VALUES.valueToSet }}"
```
