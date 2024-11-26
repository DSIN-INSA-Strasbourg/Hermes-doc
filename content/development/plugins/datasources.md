---
title: Datasources
---

## Description

A datasource plugin is simply a `AbstractDataSourcePlugin` subclass designed to link hermes-server with any datasource.

It requires methods to connect and disconnect to datasource, and to fetch, add, modify and delete data.

## Requirements

Here is a commented minimal plugin implementation that won't do anything.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Required to subclass AbstractDataSourcePlugin
from lib.plugins import AbstractDataSourcePlugin

# Required for type hints
from typing import Any

# Required to indicate to hermes which class it has to instantiate
HERMES_PLUGIN_CLASSNAME = "MyDatasourcePluginClassName"

class MyDatasourcePluginClassName(AbstractDataSourcePlugin):
    def __init__(self, settings: dict[str, Any]):
        # Instantiate new plugin and store a copy of its settings dict in self._settings
        super().__init__(settings)
        # ... plugin init code

    def open(self):
        """Establish connection with datasource"""

    def close(self):
        """Close connection with datasource"""

    def fetch(
        self,
        query: str | None,
        vars: dict[str, Any],
    ) -> list[dict[str, Any]]:
        """Fetch data from datasource with specified query and optional queryvars.
        Returns a list of dict containing each entry fetched, with REMOTE_ATTRIBUTES
        as keys, and corresponding fetched values as values"""

    def add(self, query: str | None, vars: dict[str, Any]):
        """Add data to datasource with specified query and optional queryvars"""

    def delete(self, query: str | None, vars: dict[str, Any]):
        """Delete data from datasource with specified query and optional queryvars"""

    def modify(self, query: str | None, vars: dict[str, Any]):
        """Modify data on datasource with specified query and optional queryvars"""
```

## Methods

### Connection methods

As they don't take any arguments, the `open` and `close` methods should rely on plugin settings.
For stateless datasources, they may do nothing.

### *fetch* method

This method is called to fetch some data and provide it to hermes-server.

Depending on the plugin implementation, it may rely on the `query` argument or the `vars` argument, or both.

The result must be returned as a list of dict. Each list item is a fetched entry stored in a dict, with attribute name as key, and its corresponding value. The value must be of one of the following Python types:

- `None`
- int
- float
- str
- datetime.datetime
- bytes

Allowed iterable types are:

- list
- dict

Values ​​must be of one of the types mentioned above. All other types are invalid.

### *add*, *delete*, and *modify* methods

These methods are used to modify the datasource, when possible.

Depending on the technical constraints of the data source, they can all be implemented in the same way or not.

Depending on the plugin implementation, they may rely on the `query` argument or the `vars` argument, or both.

## Error handling

No exception should be caught, to allow Hermes error handling to function properly.
