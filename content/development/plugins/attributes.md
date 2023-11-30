---
title: Attributes
---

## Description

An attribute plugin is simply an `AbstractAttributePlugin` subclass designed to implement a Jinja filter.

## Requirements

Here is a commented minimal plugin implementation that won't do anything.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Required to subclass AbstractAttributePlugin
from lib.plugins import AbstractAttributePlugin

# Required to use the Jinja Undefined state
from jinja2 import Undefined

# Required for type hints
from typing import Any

# Only needed if the plugin use logging features
import logging
logger = logging.getLogger("hermes")

# Required to indicate to hermes which class it has to instantiate
HERMES_PLUGIN_CLASSNAME = "MyPluginClassName"

class MyPluginClassName(AbstractAttributePlugin):
    def __init__(self, settings: dict[str, any]):
        # Instantiate new plugin and store a copy of its settings dict in self._settings
        super().__init__(settings)
        # ... plugin init code

    def filter(self, value: Any | None | Undefined) -> Any:
        # Filter that does nothing
        return value
```

## *filter* method

You should consider reading the official [Jinja documentation](https://jinja.palletsprojects.com/en/3.1.x/api/#writing-filters) about custom filters.

The `filter()` method always take at least one `value` parameter, and may have some other.

Its generic prototype is :

```py
def filter(self, value: Any | None | Undefined, *args: Any, **kwds: Any) -> Any:
```

In Jinja, it is called with :

```jinja
"{{ value | filter }}"
"{{ value | filter(otherarg1, otherarg2) }}"
"{{ value | filter(otherarg1=otherarg1_value, otherarg2=otherarg2_value) }}"
```

The above expression are replaced by the filter return value.

### Example : the *datetime_format* attribute plugin

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Required to subclass AbstractAttributePlugin
from lib.plugins import AbstractAttributePlugin

# Required to use the Jinja Undefined state
from jinja2 import Undefined

# Required for type hints
from typing import Any

from datetime import datetime

# Required to indicate to hermes which class it has to instantiate
HERMES_PLUGIN_CLASSNAME = "DatetimeFormatPlugin"

class DatetimeFormatPlugin(AbstractAttributePlugin):
    def filter(self, value:Any, format:str="%H:%M %d-%m-%y") -> str | Undefined:
        if isinstance(value, Undefined):
            return value

        if not isinstance(value, datetime):
            raise TypeError(f"""Invalid type '{type(value)}' for datetime_format value : must be a datetime""")

        return value.strftime(format)
```

This filter can now be called with :

```jinja
"{{ a_datetime_attribute | datetime_format }}"
"{{ a_datetime_attribute | datetime_format('%m/%d/%Y, %H:%M:%S') }}"
"{{ a_datetime_attribute | datetime_format(format='%m/%d/%Y') }}"
```
