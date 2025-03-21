---
title: Clients
---

## Description

A client plugin is simply a `GenericClient` subclass designed to implement simple events handlers, and to split their tasks in atomic subtasks to ensure consistent error reprocessing.

## Requirements

Here is a commented minimal plugin implementation that won't do anything, as it doesn't implement any event handlers yet.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Required to subclass GenericClient
from clients import GenericClient

# Required for event handlers method type hints
from lib.config import HermesConfig # only if the plugin implement an __init__() method
from lib.datamodel.dataobject import DataObject
from typing import Any

# Required to indicate to hermes which class it has to instantiate
HERMES_PLUGIN_CLASSNAME = "MyPluginClassName"

class MyPluginClassName(GenericClient):
    def __init__(self, config: HermesConfig):
        # The 'config' var must not be used nor modified by the plugin
        super().__init__(config)
        # ... plugin init code
```

## Handlers methods

### Event handlers

For each data type set up in the client datamodel, the plugin may implement a handler for each of the 5 possible event types:

- `added`: when an object is added
- `recycled`: when an object is restored from trashbin (will never be called if trashbin is disabled)
- `modified`: when an object is modified
- `trashed`: when an object is put in trashbin (will never be called if trashbin is disabled)
- `removed`: when an object is deleted

If an event is received by a client, but its handler isn't implemented, it will silently be ignored.

Each handler must be named **on_*datatypename*_*eventtypename***.

Example for a `Mydatatype` data type:

```py
    def on_Mydatatype_added(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        pass

    def on_Mydatatype_recycled(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        pass

    def on_Mydatatype_modified(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
        cachedobj: DataObject,
    ):
        pass

    def on_Mydatatype_trashed(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        cachedobj: DataObject,
    ):
        pass

    def on_Mydatatype_removed(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        cachedobj: DataObject,
    ):
        pass
```

#### Event handlers arguments

- `objkey`: the primary key of the object affected by the event

- `eventattrs`: a dictionary containing the new object attributes. Its content depends upon the event type:
  - `added` / `recycled` events: contains all object attributes names as key, and their respective values as value
  - `modified` event: always contains three keys:
    - `added`: attributes that were previously unset, but now have a value. Attribute names as key, and their respective values as value
    - `modified`: attributes that were previously set, but whose value has changed. Attribute names as key, and their respective new values as value
    - `removed`: attributes that were previously set, but now don't have a value anymore. Attribute names as key, and `None` as value
  - `trashed` / `removed` events: always an empty dict `{}`

- `newobj`: a `DataObject` instance containing all the updated values of the object affected by the event (*see [DataObject instances](#dataobject-instances) below*)

- `cachedobj`: a `DataObject` instance containing all the previous (cached) values of the object affected by the event (*see [DataObject instances](#dataobject-instances) below*)

#### DataObject instances

Each data type object can be used intuitively through a `DataObject` instance.
Let's use a simple example with this `User` object values (without a mail) from datamodel below:

```json
{
    "user_pkey": 42,
    "uid": "jdoe",
    "givenname": "John",
    "sn": "Doe"
}
```

```yaml
hermes-client:
  datamodel:
    Users:
      hermesType: SRVUsers
      attrsmapping:
        user_pkey: srv_user_id
        uid: srv_login
        givenname: srv_firstname
        sn: srv_lastname
        mail: srv_mail
```

Now, if this object is stored in a `newobj` DataObject instance:

```py
>>> newobj.getPKey()
42

>>> newobj.user_pkey
42

>>> newobj.uid
'jdoe'

>>> newobj.givenname
'John'

>>> newobj.sn
'Doe'

>>> newobj.mail
AttributeError: 'Users' object has no attribute 'mail'

>>> hasattr(newobj, 'sn')
True

>>> hasattr(newobj, 'mail')
False
```

#### Error handling

Any unhandled exception raised in an event handler will be managed by `GenericClient`, that will append the event to its error queue.
`GenericClient` will then try to process the event regularly until it succeeds, and therefore call the event handler.

But sometimes, a handler must process several operations on target. Imagine a handler like this:

```py
    def on_Mydatatype_added(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        if condition:
            operation1()  # condition is False, operation1() is not called
        operation2()  # no error occurs
        operation3()  # this one raises an exception
```

At each retry the `operation2()` function will be called again, but this is not necessarily desirable.

It is possible to divide a handler in steps by using the `currentStep` attribute inherited from `GenericClient`, to resume the retries at the failed step.

`currentStep` always starts at 0 on normal event processing. Its new values are then up to plugin implementations.

When an error occurs, the `currentStep` value is stored in the error queue with the event.  
The error queue retries will always restore the `currentStep` value before calling the event handler.

So by implementing it like below, `operation2()` will only be called once.

```py
    def on_Mydatatype_added(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        if self.currentStep == 0:
            if condition:
                operation1()  # condition is False, operation1() is not called
                # Declare that changes have been propagated on target
                self.isPartiallyProcessed = True
            self.currentStep += 1
        
        if self.currentStep == 1:
            operation2()  # no error occurs
            # Declare that changes have been propagated on target
            self.isPartiallyProcessed = True
            self.currentStep += 1

        if self.currentStep == 2:
            operation3()  # this one raises an exception
            # Declare that changes have been propagated on target
            self.isPartiallyProcessed = True
            self.currentStep += 1
```

##### Understanding `isPartiallyProcessed` attribute

The `isPartiallyProcessed` attribute inherited from `GenericClient` indicates if the current event processing has already propagated some changes on target. Therefore, it must be set to `True` as soon as the slightest modification has been propagated to the target.  
It allows autoremediation to merge events whose `currentStep` is different from 0 but whose previous steps have not modified anything on the target.

`isPartiallyProcessed` is always `False` on normal event processing. Its value change is up to plugin implementations.

With the implementation example above, and an exception raised by `operation3()`, the autoremediation would not try to merge this partially processed event with possible subsequent events, as `isPartiallyProcessed` is `True`.

With the implementation example above, but an exception raised by `operation2()`, the autoremediation would try to merge this unprocessed event with possible subsequent events, as `isPartiallyProcessed` is still `False`.

### *on_save* handler

A special handler may be implemented when hermes just have saved its cache files: once some events have been processed and no event is waiting on the message bus, or before ending.

{{% notice warning %}}
As this handler isn't a standard event handler, `GenericClient` can't handle exceptions for it, and process to a retry later.

**Any unhandled exception raised in this handler will immediately terminate the client.**

It's up to the implementation to avoid errors.
{{% /notice %}}

```py
    def on_save(self):
        pass
```

## GenericClient properties and methods

## Properties

- ```py
  currentStep: int
  ```

  Step number of current event processed. Allow clients to resume an event where it has failed.

- ```py
  isPartiallyProcessed: bool
  ```

  Indicates if the current event processing has already propagated some changes on target.  
  Must be set to `True` as soon as the slightest modification has been propagated to the target.  
  It allows autoremediation to merge events whose `currentStep` is different from 0 but whose previous steps have not modified anything on the target.

- ```py
  isAnErrorRetry: bool
  ```

  Read-only attribute that can let client plugin handler know if the current event is being processed as part of an error retry. This can be useful for example to perform additional checks when a library happens to throw exceptions even though it has correctly processed the requested changes, as python-ldap sometimes does.

- ```py
  config: dict[str, Any]
  ```

  Dict containing the client plugin configuration.

## Methods

- ```py
  def getDataobjectlistFromCache(objtype: str) -> DataObjectList
  ```

  Returns cache of specified objtype, by reference. Raise `IndexError` if `objtype` is invalid
  {{% notice warning %}}
  Any modification of the cache content will mess up your client!!!
  {{% /notice %}}

- ```py
  def getObjectFromCache(objtype: str, objpkey: Any ) -> DataObject
  ```

  Returns a deepcopy of an object from cache. Raise `IndexError` if `objtype` is invalid, or if `objpkey` is not found

- ```py
  def mainLoop() -> None
  ```

  Client main loop
  {{% notice warning %}}
  Called by Hermes, to start the client. **Must never be called nor overridden**
  {{% /notice %}}
