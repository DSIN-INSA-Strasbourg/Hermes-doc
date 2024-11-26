---
title: sqlite
---

## Description

Ce plugin permet à hermes-client de recevoir des événements depuis une base de données SQLite.

## Configuration

```yaml
hermes:
  plugins:
    messagebus:
      sqlite:
        settings:
          # OBLIGATOIRE :
          uri: /path/to/.hermes/bus.sqlite
```
