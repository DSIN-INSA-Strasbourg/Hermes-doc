---
title: Configuration
url: configuration
weight: 3
---

An hermes application will look for its YAML configuration file on current working directory.

The configuration file must be named with ***APPNAME*-config.yml**, ie :

- `hermes-server-config.yml` for *server* and *server-cli*
- `hermes-client-usersgroups_null-config.yml` for *client-usersgroups_null* and *client-usersgroups_null-cli*

Settings are separated in several YAML sections :

- [hermes](hermes) : settings shared by server and all clients
- [hermes-server](hermes-server) : server's settings
- [hermes-client](hermes-client) : settings shared by all clients
<!-- TODO : - [hermes-client-*clientName*](hermes-client-plugins) : specific client's settings -->
