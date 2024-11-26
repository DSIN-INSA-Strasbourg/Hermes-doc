---
title: Attributs
---

## Description

Un plugin d'attribut est simplement une classe fille de `AbstractAttributePlugin` conçue pour implémenter un filtre Jinja.

## Conditions requises

Voici une implémentation de plugin minimale commentée qui ne fera rien.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Nécessaire pour hériter de la classe AbstractAttributePlugin
from lib.plugins import AbstractAttributePlugin

# Nécessaire pour utiliser le statut Jinja Undefined
from jinja2 import Undefined

# Nécessaire pour les annotations de type
from typing import Any

# Nécessaire pour indiquer à hermes quelle classe il devra instancier
HERMES_PLUGIN_CLASSNAME = "MyPluginClassName"

class MyPluginClassName(AbstractAttributePlugin):
    def __init__(self, settings: dict[str, any]):
        # Crée une nouvelle instance du plugin et stocke une copie de
        # son dictionnaire de paramètres dans self._settings
        super().__init__(settings)
        # ... code d'initialisation du plugin

    def filter(self, value: Any | None | Undefined) -> Any:
        # Filtre qui ne fait rien
        return value
```

## Méthode *filter*

Vous devriez consulter la [documentation officielle de Jinja](https://jinja.palletsprojects.com/en/3.1.x/api/#writing-filters) sur les filtres personnalisés.

La méthode `filter()` prend toujours au moins un paramètre `value` et peut en avoir d'autres.

Son prototype générique est :

```py
def filter(self, value: Any | None | Undefined, *args: Any, **kwds: Any) -> Any:
```

En Jinja, on l'utilise ainsi :

```jinja
"{{ value | filter }}"
"{{ value | filter(otherarg1, otherarg2) }}"
"{{ value | filter(otherarg1=otherarg1_value, otherarg2=otherarg2_value) }}"
```

Les expressions ci-dessus seront remplacées par la valeur de retour du filtre.

### Exemple : le plugin d'attribut *datetime_format*

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Nécessaire pour hériter de la classe AbstractAttributePlugin
from lib.plugins import AbstractAttributePlugin

# Nécessaire pour utiliser le statut Jinja Undefined
from jinja2 import Undefined

# Nécessaire pour les annotations de type
from typing import Any

from datetime import datetime

# Nécessaire pour indiquer à hermes quelle classe il devra instancier
HERMES_PLUGIN_CLASSNAME = "DatetimeFormatPlugin"

class DatetimeFormatPlugin(AbstractAttributePlugin):
    def filter(self, value:Any, format:str="%H:%M %d-%m-%y") -> str | Undefined:
        if isinstance(value, Undefined):
            return value

        if not isinstance(value, datetime):
            raise TypeError(f"""Invalid type '{type(value)}' for datetime_format value: must be a datetime""")

        return value.strftime(format)
```

Le filtre peut désormais être utilisé ainsi :

```jinja
"{{ a_datetime_attribute | datetime_format }}"
"{{ a_datetime_attribute | datetime_format('%m/%d/%Y, %H:%M:%S') }}"
"{{ a_datetime_attribute | datetime_format(format='%m/%d/%Y') }}"
```
