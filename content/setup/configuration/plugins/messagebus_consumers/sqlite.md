---
title: sqlite
---

## Description

This plugin allows hermes-client to receive events from an SQLite database.

## Configuration

```yaml
hermes:
  plugins:
    messagebus:
      sqlite:
        settings:
          # MANDATORY:
          uri: /path/to/.hermes/bus.sqlite
```
