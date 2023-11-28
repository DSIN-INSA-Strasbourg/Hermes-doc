---
title: Messagebus consumers
---

## Description

A datsource plugin is simply a `AbstractMessageBusConsumerPlugin` subclass designed to link hermes-client with any message bus.

It requires methods to connect and disconnect to message bus, and to consume available events.

### Features required from message bus

- Allow to specify a message key/category (*producers*) and to filter message of a specified key/category (*consumers*)
- Allow to consume a same message more than once
- Implementing a message offset, allowing consumers to seek at next required message. As it will be stored in clients cache, this offset must be of one of the Python types below :
  - int
  - float
  - str
  - bytes

## Requirements

Here is a commented minimal plugin implementation that won't do anything.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Required to subclass AbstractMessageBusConsumerPlugin
from lib.plugins import AbstractMessageBusConsumerPlugin
# Required to return Event
from lib.datamodel.event import Event

# Required for type hints
from typing import Any, Iterable

# Only needed if the plugin use logging features
import logging
logger = logging.getLogger("hermes")

# Required to indicate to hermes which class it has to instanciate
HERMES_PLUGIN_CLASSNAME = "MyMessagebusConsumerPluginClassName"

class MyMessagebusConsumerPluginClassName(AbstractMessageBusConsumerPlugin):
    def __init__(self, settings: dict[str, Any]):
        # Instanciate new plugin and store a copy of its settings dict in self._settings
        super().__init__(settings)
        # ... plugin init code

    def open(self) -> Any:
        """Establish connection with messagebus"""

    def close(self):
        """Close connection with messagebus"""

    def seekToBeginning(self):
        """Seek to first (older) event in message bus queue"""

    def seek(self, offset: Any):
        """Seek to specified offset event in message bus queue"""

    def setTimeout(self, timeout_ms: int | None):
        """Set timeout (in milliseconds) before aborting when waiting for next event.
        If None, wait forever"""

    def findNextEventOfCategory(self, category: str) -> Event | None:
        """Lookup for first message with specified category and returns it,
        or returns None if none was found"""

    def __iter__(self) -> Iterable:
        """Iterate over message bus returning each Event, starting at current offset.
        When every event has been consumed, wait for next message until timeout set with
        setTimeout() has been reached"""
```

## Methods to implement

### Connection methods

As they don't take any arguments, the `open` and `close` methods should rely on plugin settings.

### *seekToBeginning* method

Seek to first (older) event in message bus queue.

### *seek* method

Seek to specified offset event in message bus queue.

### *setTimeout* method

Set timeout (in milliseconds) before aborting when waiting for next event. If `None`, wait forever.

### *findNextEventOfCategory* method

Lookup for first message with specified category and returns it, or returns `None` if none was found.

As this method will browse the message bus, the current offset will be modified.

### *\_\_iter\_\_* method

Returns an Iterable that will yield all events available on message bus, starting from current offset.

Those unserializable attributes of `Event` instance must be defined before yielding it :

- `offset` (*int* | *float* | *str* | *bytes*) : offset of the event in message bus
- `timestamp` (*dattime.datetime*) : timestamp of the event

## Event properties and methods

### Methods

- ```py
  @staticmethod
  def from_json(jsondata: str | dict[Any, Any]) -> Event
  ```

  Deserialize a json string or dict to a new Event instance, and returns it
