---
title: Installation
url: installation
weight: 2
---

## Requirements

- [Python](https://www.python.org/) >= 3.10 with [pip](https://pip.pypa.io/en/stable/)
- Run on Linux (required for CLI that uses Unix socket)
- A message bus server, ie. [Apache Kafka](https://kafka.apache.org/).
- [direnv](https://direnv.net/) - only if you wish to use the `reset_venv` helper script

{{% notice tip %}}
For testing, Hermes provides a simple SQLite message bus implementation, but it shouldn't be used for production
{{% /notice %}}

## Installation

1. Download and extract the [hermes latest](https://github.com/DSIN-INSA-Strasbourg/Hermes/archive/refs/tags/Hermes-latest.zip) archive

2. (*Optional*) If you want to minimize install footprint, you may remove all unnecessary plugins by deleting their directory in :

    - `clients/`
    - `plugins/attributes/`
    - `plugins/datasources/`
    - `plugins/messagebus_consumers/`
    - `plugins/messagebus_producers/`

    If your installation is for running **hermes-server** only (without clients), you may remove the following directories :
    - `clients`
    - `plugins/messagebus_consumers`

    If your installation is for running one or more **hermes-client** only (without server), you may remove the following directories :
    - `server`
    - `plugins/datasources`
    - `plugins/messagebus_producers`

3. Set up a venv and install all requirements

    - Automatically with the provided script `./reset_venv`
    - Manually. You can generate and install python requirements with the following commands :

      ```bash
      cat "requirements.txt" "clients/"*"/requirements.txt" "plugins/"*/*"/requirements.txt" > all_requirements.txt 2>/dev/null

      pip3 install -r all_requirements.txt
      ```

## Configuration

Please refer to [Configuration](/en/configuration/) section of this guide.

## Running

You may start any hermes app (server, server-cli, client, client-cli) directly with the `hermes.py` launcher, specifying the app name as first argument, or by a symlink.

In either cases, the configuration will be searched in current working directory.

### Running from launcher

```bash
# Server
/path/to/hermes.py server
# Server CLI
/path/to/hermes.py server-cli

# Client usersgroups_null
/path/to/hermes.py client-usersgroups_null
# Client usersgroups_null CLI
/path/to/hermes.py client-usersgroups_null-cli
```

### Running using symlinks

If you prefer to avoid using hermes app name as first argument, you may symlink `hermes.py` to `hermes-`*appname*, ie

```bash
ln -s hermes.py hermes-server
ln -s hermes.py hermes-server-cli
ln -s hermes.py hermes-client-usersgroups_null
ln -s hermes.py hermes-client-usersgroups_null-cli
# ...
```

and running them with :

```bash
# Server
/path/to/hermes-server
# Server CLI
/path/to/hermes-server-cli

# Client usersgroups_null
/path/to/hermes-client-usersgroups_null
# Client usersgroups_null CLI
/path/to/hermes-client-usersgroups_null-cli
```