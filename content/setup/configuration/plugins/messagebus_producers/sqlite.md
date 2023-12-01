---
title: sqlite
---

## Description

This plugin allows hermes-server to send produced events over an SQLite database.

{{% notice warning %}}
This plugin is provided for testing but shouldn’t be used for production.
{{% /notice %}}

## Configuration

To emulate the behavior of other message buses, that delete messages once some conditions are met, `retention_in_days` can be set. It will delete messages older than the specified number of days.

```yaml
hermes:
  plugins:
    messagebus:
      sqlite:
        settings:
          # MANDATORY:
          uri: /path/to/.hermes/bus.sqlite
          retention_in_days: 1
```
