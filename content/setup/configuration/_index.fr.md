---
title: Configuration
weight: 3
---

Une application hermes recherchera son fichier de configuration YAML dans le répertoire de travail courant.

Le fichier de configuration doit être nommé ***APPNAME*-config.yml**, *e.g.* :

- `hermes-server-config.yml` pour *server* et *server-cli*
- `hermes-client-usersgroups_null-config.yml` pour *client-usersgroups_null* et *client-usersgroups_null-cli*

Les paramètres sont séparés en plusieurs sections YAML :

- [hermes](/setup/configuration/hermes/) : paramètres partagés par le serveur et tous les clients
- [hermes-server](/setup/configuration/hermes-server/) : paramètres du serveur
- [hermes-client](/setup/configuration/hermes-client/) : paramètres partagés par tous les clients
- [hermes-client-*clientName*](/setup/configuration/plugins/hermes-client/) : paramètres spécifiques à un plugin client
- hermes
  - plugins
    - [attributes](/setup/configuration/plugins/attributes/) : paramètres des plugins d'attribut
    - [datasources](/setup/configuration/plugins/datasources/) : paramètres des plugins de source de données
    - [messagebus](/setup/configuration/plugins/messagebus_producers/) : paramètres des plugins producteur de bus de messages
    - [messagebus](/setup/configuration/plugins/messagebus_consumers/) : paramètres des plugins consommateur de bus de messages

Pour des raisons de sécurité, il peut être souhaitable d'autoriser certains utilisateurs à utiliser la CLI sans leur accorder un accès en lecture au fichier de configuration. Pour ce faire, il suffit simplement de créer un fichier de configuration CLI facultatif nommé ***APPNAME*-cli-config.yml**, *e.g.* :

- `hermes-server-cli-config.yml` pour *server-cli*
- `hermes-client-usersgroups_null-cli-config.yml` pour *client-usersgroups_null-cli*

Ce fichier ne doit contenir que les directives suivantes :

```yaml
hermes:
  cli_socket:
    path: /path/to/cli/sockfile.sock
```
