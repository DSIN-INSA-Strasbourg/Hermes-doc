---
title: Sources de données
---

## Description

Un plugin de source de données est simplement une classe fille de `AbstractDataSourcePlugin` conçue pour permettre à hermes-server d'interroger n'importe quelle source de données.

Il faudra y implémenter des méthodes pour se connecter et se déconnecter de la source de données, et pour récupérer, ajouter, modifier et supprimer des données.

## Conditions requises

Voici une implémentation de plugin minimale commentée qui ne fera rien.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Nécessaire pour hériter de la classe AbstractDataSourcePlugin
from lib.plugins import AbstractDataSourcePlugin

# Nécessaire pour les annotations de type
from typing import Any

# Nécessaire pour indiquer à hermes quelle classe il devra instancier
HERMES_PLUGIN_CLASSNAME = "MyDatasourcePluginClassName"

class MyDatasourcePluginClassName(AbstractDataSourcePlugin):
    def __init__(self, settings: dict[str, Any]):
        # Crée une nouvelle instance du plugin et stocke une copie de
        # son dictionnaire de paramètres dans self._settings
        super().__init__(settings)
        # ... code d'initialisation du plugin

    def open(self):
        """Établit la connexion avec la source de données"""

    def close(self):
        """Ferme la connexion avec la source de données"""

    def fetch(
        self,
        query: str | None,
        vars: dict[str, Any],
    ) -> list[dict[str, Any]]:
        """Récupère des données à partir de la source de données avec la requête
        spécifiée et/ou des variables.
        Renvoie une liste de dictionnaires contenant chaque entrée extraite, avec
        REMOTE_ATTRIBUTES comme clés et les valeurs extraites correspondantes
        comme valeurs"""

    def add(self, query: str | None, vars: dict[str, Any]):
        """Ajoute des données à la source de données avec la requête spécifiée
        et/ou des variables"""

    def delete(self, query: str | None, vars: dict[str, Any]):
        """Supprime des données de la source de données avec la requête spécifiée
        et/ou des variables"""

    def modify(self, query: str | None, vars: dict[str, Any]):
        """Modifie des données sur la source de données avec la requête spécifiée
        et/ou des variables"""
```

## Méthodes

### Méthodes de connexion

Comme elles ne prennent aucun argument, les méthodes `open` et `close` doivent s'appuyer sur les paramètres du plugin.
Pour les sources de données sans état, elles peuvent ne rien faire.

### Méthode *fetch*

Cette méthode est appelée pour récupérer des données et les fournir à hermes-server.

Selon l'implémentation du plugin, elle peut s'appuyer sur l'argument `query`, l'argument `vars`, ou les deux.

Le résultat doit être renvoyé sous forme de liste de dictionnaires. Chaque élément de la liste est une entrée récupérée stockée dans un dictionnaire, avec le nom de l'attribut comme clé et sa valeur correspondante comme valeur. La valeur doit être de l'un des types Python suivants :

- `None`
- int
- float
- str
- datetime.datetime
- bytes

Les types itérables autorisés sont :

- list
- dict

Les valeurs doivent impérativement être d'un des types mentionnés ci-dessus. Tous les autres types sont invalides.

### Méthodes *add*, *delete*, et *modify*

Ces méthodes permettent de modifier le contenu de la source de données, lorsque cela est possible.

Selon les contraintes techniques de la source de données, elles peuvent toutes être implémentées de la même manière ou non.

Selon l'implémentation du plugin, elles peuvent s'appuyer sur l'argument `query`, l'argument `vars`, ou les deux.

## Gestion d'erreur

Aucune exception ne devrait être interceptée afin de permettre à la gestion d'erreur d'Hermes de fonctionner correctement.
