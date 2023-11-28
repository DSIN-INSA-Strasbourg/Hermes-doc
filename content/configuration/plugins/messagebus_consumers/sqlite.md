---
title: sqlite
url: configuration/plugins/messagebus_consumers/sqlite
---

## Description

This plugin allow hermes-client to receive events from a SQLite database.

{{% notice warning %}}
This plugin is provided for testing, but shouldn’t be used for production.
{{% /notice %}}

## Configuration

```yaml
hermes:
  plugins:
    messagebus:
      sqlite:
        settings:
          # MANDATORY :
          uri: /path/to/.hermes/bus.sqlite
```
