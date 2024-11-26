---
title: Exécution
weight: 4
---

Vous pouvez démarrer n'importe quelle application Hermes (server, server-cli, client, client-cli) directement avec le lanceur `hermes.py`, en spécifiant le nom de l'application comme premier argument, ou par un lien symbolique.

Dans les deux cas, la configuration sera recherchée dans le répertoire de travail courant.

## Exécution depuis le lanceur

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

## Exécution depuis un lien symbolique

Si vous souhaitez éviter de spécifier le nom de l'application hermes comme premier argument, il est possible de créer un lien symbolique de `hermes.py` vers `hermes-`*appname*, *e.g.*:

```bash
ln -s hermes.py hermes-server
ln -s hermes.py hermes-server-cli
ln -s hermes.py hermes-client-usersgroups_null
ln -s hermes.py hermes-client-usersgroups_null-cli
# ...
```

et de l'exécuter ainsi :

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

## Arguments des commandes

Le serveur et les clients n'acceptent aucun argument, car ils sont conçus pour être contrôlés via la CLI.

Une fois le serveur ou le client démarré, vous pouvez demander la liste des commandes CLI disponibles avec l'option `-h` ou `--help`.

Pour le serveur :

```bash
$ ./hermes.py server-cli -h
usage: hermes-server-cli [-h] {initsync,update,quit,pause,resume,status} ...

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

Pour un client :

```bash
$ ./hermes.py client-usersgroups_null-cli -h
usage: hermes-client-usersgroups_null-cli [-h] {quit,pause,resume,status} ...

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
