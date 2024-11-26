---
title: Consommateurs de bus de messages
---

## Description

Un plugin consommateur de bus de messages est simplement une classe fille de `AbstractMessageBusConsumerPlugin` conçue pour permettre à hermes-client d'interroger n'importe quel bus de messages.

Il faudra y implémenter des méthodes pour se connecter et se déconnecter au bus de messages, et pour consommer les événements disponibles.

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

# Nécessaire pour hériter de la classe AbstractMessageBusConsumerPlugin
from lib.plugins import AbstractMessageBusConsumerPlugin
# Required to return Event
from lib.datamodel.event import Event

# Nécessaire pour les annotations de type
from typing import Any, Iterable

# Nécessaire pour indiquer à hermes quelle classe il devra instancier
HERMES_PLUGIN_CLASSNAME = "MyMessagebusConsumerPluginClassName"

class MyMessagebusConsumerPluginClassName(AbstractMessageBusConsumerPlugin):
    def __init__(self, settings: dict[str, Any]):
        # Crée une nouvelle instance du plugin et stocke une copie de
        # son dictionnaire de paramètres dans self._settings
        super().__init__(settings)
        # ... code d'initialisation du plugin

    def open(self) -> Any:
        """Établit la connexion avec le bus de message"""

    def close(self):
        """Ferme la connexion avec le bus de message"""

    def seekToBeginning(self):
        """Rechercher le premier événement (le plus ancien) dans la file du bus
        de messages"""

    def seek(self, offset: Any):
        """Rechercher l'événement de l'offset spécifié dans la file du bus de
        messages"""

    def setTimeout(self, timeout_ms: int | None):
        """Définit le délai d'attente (en millisecondes) avant d'interrompre
        l'attente du prochain événement. Si None, attend indéfiniment"""

    def findNextEventOfCategory(self, category: str) -> Event | None:
        """Recherche le premier message avec la catégorie spécifiée et le renvoie,
        ou renvoie None si aucun n'a été trouvé"""

    def __iter__(self) -> Iterable:
        """Itère sur le bus de messages en renvoyant chaque événement, en
        commençant à l'offset courant.
        Lorsque chaque événement a été consommé, attends le message suivant
        jusqu'à ce que le délai d'expiration défini avec setTimeout() soit
        atteint"""
```

## Méthodes à implémenter

### Méthodes de connexion

Comme elles ne prennent aucun argument, les méthodes `open` et `close` doivent s'appuyer sur les paramètres du plugin.

### Méthode *seekToBeginning*

Rechercher le premier événement (le plus ancien) dans la file du bus de messages.

### Méthode *seek*

Rechercher l'événement de l'offset spécifié dans la file du bus de messages.

### Méthode *setTimeout*

Définit le délai d'attente (en millisecondes) avant d'interrompre l'attente du prochain événement. Si `None`, attend indéfiniment.

### Méthode *findNextEventOfCategory*

Recherche le premier message avec la catégorie spécifiée et le renvoie, ou renvoie `None` si aucun n'a été trouvé.

Comme cette méthode parcourt le bus de messages, l'offset courant sera modifié.

### Méthode *\_\_iter\_\_*

Renvoie un Iterable qui génère (*yield*) tous les événements disponibles sur le bus de messages, à partir de l'offset courant.

Ces attributs non-sérialisables de l'instance `Event` doivent être définis avant de le générer (*yield*) :

- `offset` (*int* | *float* | *str* | *bytes*) : offset de l'événement dans le bus de messages
- `timestamp` (*dattime.datetime*) : horodatage de l'événement

## Propriétés et méthodes de la classe Event

### Méthodes

- ```py
  @staticmethod
  def from_json(jsondata: str | dict[Any, Any]) -> Event
  ```

  Désérialise une chaîne ou un dictionnaire JSON en une nouvelle instance Event et la renvoie
