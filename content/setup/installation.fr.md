---
title: Installation
weight: 2
---

## Configuration requise

- [Python](https://www.python.org/) 3.10, 3.11 ou 3.12 avec [pip](https://pip.pypa.io/en/stable/)
- Exécution sous Linux (requis pour la CLI qui utilise un socket Unix)
- Un serveur de bus de messages, *e.g.* [Apache Kafka](https://kafka.apache.org/) - recommandé mais une implémentation [sqlite](/setup/configuration/plugins/messagebus_producers/sqlite/) est fournie
- [direnv](https://direnv.net/) - uniquement si vous souhaitez utiliser le script d'aide `reset_venv`

## Guide d'installation

1. Télécharger et extraire l'archive [hermes latest](https://github.com/DSIN-INSA-Strasbourg/Hermes/archive/refs/heads/main.zip)

2. (*Facultatif*) Si vous souhaitez réduire l'empreinte de l'installation, vous pouvez supprimer le répertoire `tests`, le fichier `tox.ini` et tous les plugins inutiles en supprimant leurs répertoires dans :

    - `plugins/attributes/`
    - `plugins/clients/`
    - `plugins/datasources/`
    - `plugins/messagebus_consumers/`
    - `plugins/messagebus_producers/`

    Si votre installation est destinée à exécuter **hermes-server** uniquement (sans clients), vous pouvez supprimer les répertoires suivants :
    - `clients`
    - `plugins/clients/`
    - `plugins/messagebus_consumers`

    Si votre installation est destinée à exécuter un ou plusieurs **hermes-client** uniquement (sans serveur), vous pouvez supprimer les répertoires suivants :
    - `server`
    - `plugins/datasources`
    - `plugins/messagebus_producers`

3. Configurer un venv et installer tous les pré-requis

    - Automatiquement avec le script fourni `./reset_venv`
    - Manuellement. Vous pouvez générer et installer les pré-requis Python avec les commandes suivantes :

    ```bash
    cat "requirements.txt" "plugins/"*/*"/requirements.txt" > all_requirements.txt 2>/dev/null

    pip3 install -r all_requirements.txt
    ```
