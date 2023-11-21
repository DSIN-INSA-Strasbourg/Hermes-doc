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

1. Download and extract the [hermes latest](https://github.com/DSIN-INSA-Strasbourg/Hermes/archive/refs/heads/main.zip) archive

2. (*Optional*) If you want to minimize install footprint, you may remove `tests` directory, tests launcher `run_tests.sh` and all unnecessary plugins by deleting their directory in :

    - `plugins/attributes/`
    - `plugins/clients/`
    - `plugins/datasources/`
    - `plugins/messagebus_consumers/`
    - `plugins/messagebus_producers/`

    If your installation is for running **hermes-server** only (without clients), you may remove the following directories :
    - `clients`
    - `plugins/clients/`
    - `plugins/messagebus_consumers`

    If your installation is for running one or more **hermes-client** only (without server), you may remove the following directories :
    - `server`
    - `plugins/datasources`
    - `plugins/messagebus_producers`

3. Set up a venv and install all requirements

    - Automatically with the provided script `./reset_venv`
    - Manually. You can generate and install python requirements with the following commands :

      ```bash
      cat "requirements.txt" "plugins/"*/*"/requirements.txt" > all_requirements.txt 2>/dev/null

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

### Commands arguments

The server and clients doesn't take any arguments, as they're designed to be controlled over the CLI.

Once the server or client is started, you may ask for available CLI commands with `-h` or `--help` option.

For server :

```bash
$ ./hermes.py server-cli -h
usage: hermes.py [-h] {initsync,update,quit,pause,resume,status} ...

Hermes Server CLI

positional arguments:
  {initsync,update,quit,pause,resume,status}
                        Sub-commands
    initsync            Send specific init message containing all data but passwords. Useful to fill new client
    update              Force update now, ignoring updateInterval
    quit                Stop server
    pause               Pause processing until 'resume' command is sent
    resume              Resume processing that has been paused with 'pause'
    status              Show server status

options:
  -h, --help            show this help message and exit
```

For a client :

```bash
$ ./hermes.py client-usersgroups_null-cli -h
usage: hermes.py [-h] {quit,pause,resume,status} ...

Hermes client hermes-client-usersgroups_null CLI

positional arguments:
  {quit,pause,resume,status}
                        Sub-commands
    quit                Stop hermes-client-usersgroups_null
    pause               Pause processing until 'resume' command is sent
    resume              Resume processing that has been paused with 'pause'
    status              Show hermes-client-usersgroups_null status

options:
  -h, --help            show this help message and exit
```
