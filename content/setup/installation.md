---
title: Installation
weight: 2
---

## Requirements

- [Python](https://www.python.org/) 3.10, 3.11, 3.12 or 3.13 with [pip](https://pip.pypa.io/en/stable/)
- Run on Linux (required for CLI that uses Unix socket)
- A message bus server, *e.g.* [Apache Kafka](https://kafka.apache.org/) - recommended but an [sqlite](/setup/configuration/plugins/messagebus_producers/sqlite/) implementation is provided
- [direnv](https://direnv.net/) - only if you wish to use the `reset_venv` helper script

## Install guide

1. Download and extract the [hermes latest](https://github.com/DSIN-INSA-Strasbourg/Hermes/archive/refs/heads/main.zip) archive

2. (*Optional*) If you want to minimize install footprint, you may remove `tests` directory, `tox.ini` file and all unnecessary plugins by deleting their directory in:

    - `plugins/attributes/`
    - `plugins/clients/`
    - `plugins/datasources/`
    - `plugins/messagebus_consumers/`
    - `plugins/messagebus_producers/`

    If your installation is for running **hermes-server** only (without clients), you may remove the following directories:
    - `clients`
    - `plugins/clients/`
    - `plugins/messagebus_consumers`

    If your installation is for running one or more **hermes-client** only (without server), you may remove the following directories:
    - `server`
    - `plugins/datasources`
    - `plugins/messagebus_producers`

3. Set up a venv and install all requirements

    - Automatically with the provided script `./reset_venv`
    - Manually. You can generate and install python requirements with the following commands:

      ```bash
      cat "requirements.txt" "plugins/"*/*"/requirements.txt" > all_requirements.txt 2>/dev/null

      pip3 install -r all_requirements.txt
      ```
