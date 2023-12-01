---
title: oracle
---

## Description

This plugin allows using an Oracle database as datasource.

## Configuration

Connection settings are required in plugin configuration.

```yaml
hermes:
  plugins:
    datasources:
      # Source name. Use whatever you want. Will be used in datamodel
      your_source_name:
        type: oracle
        settings:
          # MANDATORY: the database server DNS name or IP address
          server: dummy.example.com
          # MANDATORY: the database connection port
          port: 1234
          # MANDATORY: the database SID
          sid: DUMMY
          # MANDATORY: the database credentials to use
          login: HERMES_DUMMY
          password: "DuMmY_p4s5w0rD"
```

## Usage

Specify a query. If you'd like to provide values from cache, you should provide them in a `vars` dict, and refer to them by specifying the column-prefixed `:` var key name in the query: this will automatically sanitize the query.

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
              FROM AN_ORACLE_TABLE

          commit_one:
            type: modify
            query: >-
              UPDATE AN_ORACLE_TABLE
              SET
                valueToSet = :sanitized_valueToSet
              WHERE pkey = :sanitized_pkey

            vars:
              sanitized_pkey: "{{ ITEM_FETCHED_VALUES.pkey }}"
              sanitized_valueToSet: "{{ ITEM_FETCHED_VALUES.valueToSet }}"
```
