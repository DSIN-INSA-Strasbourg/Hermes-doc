---
title: Messagebus producers
---

## Description

A messagebus producer plugin is simply a `AbstractMessageBusProducerPlugin` subclass designed to link hermes-server with any message bus.

It requires methods to connect and disconnect to message bus, and to produce (send) events over it.

### Features required from message bus

- Allow to specify a message key/category (*producers*) and to filter message of a specified key/category (*consumers*)
- Allow to consume a same message more than once
- Implementing a message offset, allowing consumers to seek the next required message. As it will be stored in clients cache, this offset must be of one of the Python types below:
  - int
  - float
  - str
  - bytes

## Requirements

Here is a commented minimal plugin implementation that won't do anything.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Required to subclass AbstractMessageBusProducerPlugin
from lib.plugins import AbstractMessageBusProducerPlugin

# Required for type hints
from lib.datamodel.event import Event
from typing import Any

# Required to indicate to hermes which class it has to instantiate
HERMES_PLUGIN_CLASSNAME = "MyMessagebusProducerPluginClassName"

class MyMessagebusProducerPluginClassName(AbstractMessageBusProducerPlugin):
    def __init__(self, settings: dict[str, Any]):
        # Instantiate new plugin and store a copy of its settings dict in self._settings
        super().__init__(settings)
        # ... plugin init code

    def open(self) -> Any:
        """Establish connection with messagebus"""

    def close(self):
        """Close connection with messagebus"""

    def _send(self, event: Event):
        """Send specified event to message bus"""
```

## Methods to implement

### Connection methods

As they don't take any arguments, the `open` and `close` methods should rely on plugin settings.

### *_send* method

{{% notice note %}}
Be careful to overload the `_send()` method and not the `send()` one.

The `send()` method is a wrapper that handles exceptions while calling `_send()`.
{{% /notice %}}

Send a message containing the specified event.

The consumer will require the following properties:

- `evcategory` (*str*): Key/category of the event (*stored in the Event*)
- `timestamp` (*dattime.datetime*): timestamp of the event
- `offset` (*int* | *float* | *str* | *bytes*): offset of the event in message bus

See [Event properties and methods](#event-properties-and-methods) below.

## Event properties and methods

### Properties

- ```py
  evcategory: str
  ```

  Key/category to apply to the message

### Methods

- ```py
  def to_json() -> str
  ```

  Serialize event to a json string that can be used later to be deserialized in a new Event instance
