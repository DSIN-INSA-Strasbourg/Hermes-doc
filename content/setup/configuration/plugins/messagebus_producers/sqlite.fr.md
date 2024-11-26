---
title: sqlite
---

## Description

Ce plugin permet à hermes-server d'envoyer les événements produits vers une base de données SQLite.

## Configuration

Pour imiter le comportement d'autres bus de messages qui suppriment les messages une fois certaines conditions remplies, `retention_in_days` peut être définie. Les messages plus anciens que le nombre de jours spécifié seront automatiquement supprimés.

```yaml
hermes:
  plugins:
    messagebus:
      sqlite:
        settings:
          # OBLIGATOIRE :
          uri: /path/to/.hermes/bus.sqlite
          retention_in_days: 1
```
