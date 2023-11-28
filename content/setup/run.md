---
title: Run
weight: 4
---

You may start any hermes app (server, server-cli, client, client-cli) directly with the `hermes.py` launcher, specifying the app name as first argument, or by a symlink.

In either cases, the configuration will be searched in current working directory.

## Running from launcher

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

## Running using symlinks

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

## Commands arguments

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
