---
title: Configuration
weight: 3
---

An hermes application will look for its YAML configuration file on current working directory.

The configuration file must be named with ***APPNAME*-config.yml**, *e.g.*:

- `hermes-server-config.yml` for *server* and *server-cli*
- `hermes-client-usersgroups_null-config.yml` for *client-usersgroups_null* and *client-usersgroups_null-cli*

Settings are separated in several YAML sections:

- [hermes](./hermes/): settings shared by server and all clients
- [hermes-server](./hermes-server/): server settings
- [hermes-client](./hermes-client/): settings shared by all clients
- [hermes-client-*clientName*](./plugins/hermes-client/): specific client plugin settings
- hermes
  - plugins
    - [attributes](./plugins/attributes/): attributes plugins settings
    - [datasources](./plugins/datasources/): datasources plugins settings
    - [messagebus](./plugins/messagebus_producers/): server messagebus_producers plugins settings
    - [messagebus](./plugins/messagebus_consumers/): client messagebus_consumers plugins settings

For security reasons, it may be desirable to allow certain users to use the CLI without granting them read access to the configuration file. To do this, simply create an optional CLI configuration file named ***APPNAME*-cli-config.yml**, *e.g.*:

- `hermes-server-cli-config.yml` for and *server-cli*
- `hermes-client-usersgroups_null-cli-config.yml` for *client-usersgroups_null-cli*

This file should only contain the following directives :

```yaml
hermes:
  cli_socket:
    path: /path/to/cli/sockfile.sock
```
