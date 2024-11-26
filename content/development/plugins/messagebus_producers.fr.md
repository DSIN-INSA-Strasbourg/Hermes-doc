---
title: Producteurs de bus de messages
---

## Description

Un plugin producteurs de bus de messages est simplement une classe fille de `AbstractMessageBusProducerPlugin` conçue pour permettre à hermes-server d'émettre des événements vers n'importe quel bus de messages.

Il faudra y implémenter des méthodes pour se connecter et se déconnecter au bus de messages, et pour y produire (émettre) des événements.

### Fonctionnalités requises du bus de messages

- Permettre de spécifier une clé/catégorie de message (*producteurs*) et de filtrer les messages d'une clé/catégorie spécifiée (*consommateurs*)
- Permettre de consommer un même message plusieurs fois
- Implémenter un offset de message, permettant aux consommateurs de rechercher le prochain message attendu. Comme il sera stocké dans le cache des clients, cet offset doit être de l'un des types Python ci-dessous :
  - int
  - float
  - str
  - bytes

## Conditions requises

Voici une implémentation de plugin minimale commentée qui ne fera rien.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Nécessaire pour hériter de la classe AbstractMessageBusProducerPlugin
from lib.plugins import AbstractMessageBusProducerPlugin

# Nécessaire pour les annotations de type
from lib.datamodel.event import Event
from typing import Any

# Nécessaire pour indiquer à hermes quelle classe il devra instancier
HERMES_PLUGIN_CLASSNAME = "MyMessagebusProducerPluginClassName"

class MyMessagebusProducerPluginClassName(AbstractMessageBusProducerPlugin):
    def __init__(self, settings: dict[str, Any]):
        # Crée une nouvelle instance du plugin et stocke une copie de
        # son dictionnaire de paramètres dans self._settings
        super().__init__(settings)
        # ... code d'initialisation du plugin

    def open(self) -> Any:
        """Établit la connexion avec le bus de message"""

    def close(self):
        """Ferme la connexion avec le bus de message"""

    def _send(self, event: Event):
        """Émet l'événement spécifié sur le bus de message"""
```

## Méthodes à implémenter

### Méthodes de connexion

Comme elles ne prennent aucun argument, les méthodes `open` et `close` doivent s'appuyer sur les paramètres du plugin.

### Méthode *_send*

{{% notice note %}}
Attention à surcharger la méthode `_send()` et non pas `send()`.

La méthode `send()` est un wrapper qui gère les exceptions lors de l'appel à `_send()`.
{{% /notice %}}

Envoie un message contenant l'événement spécifié.

Le consommateur aura besoin des propriétés suivantes :

- `evcategory` (*str*) : clé/catégorie de l'événement (*stockée dans l'événement*)
- `timestamp` (*dattime.datetime*) : horodatage de l'événement
- `offset` (*int* | *float* | *str* | *bytes*) : offset de l'événement dans le bus de messages

Voir [Propriétés et méthodes de la classe Event](#event-properties-and-methods) ci-dessous.

## Propriétés et méthodes de la classe Event {#event-properties-and-methods}

### Propriétés

- ```py
  evcategory: str
  ```

  Clé/catégorie à attribuer au message

### Méthodes

- ```py
  def to_json() -> str
  ```

  Sérialise un événement dans une chaîne JSON qui pourra être utilisée ultérieurement pour être désérialisée dans une nouvelle instance Event
