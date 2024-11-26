---
title: Mise en route
weight: 1
---

1. Identifiez vos prérequis :
    - la ou les sources de données à utiliser et leur type, ainsi que les données que vous souhaitez capturer sur chacune d'elles. Une fois cela fait, identifiez si le ou les [plugins de source de données](/setup/configuration/plugins/datasources/) correspondants existent
    - choisissez (et éventuellement installez) le bus de messages que vous utiliserez
    - identifiez le ou les [plugins hermes-client](/setup/configuration/plugins/hermes-client/) dont vous aurez besoin

2. Installez Hermes en suivant la section [Installation](/setup/installation/)

3. Configurez hermes-server en suivant les sections suivantes :
    - [hermes](/setup/hermes/) pour les paramètres globaux
    - configurez votre [plugin producteur de bus de messages](/setup/configuration/plugins/messagebus_producers/)
    - configurez votre ou vos [plugins de source de données](/setup/configuration/plugins/datasources/)
    - configurez vos [plugins d'attribut](/setup/configuration/plugins/attributes/), le cas échéant
    - [hermes-server](/setup/configuration/hermes-server/) pour les paramètres hermes-server et configurer votre modèle de données de serveur

4. Exécutez hermes-server en suivant la section [Exécution](/setup/run/) et une fois qu'il a effectué avec succès sa première collecte de données, générez une séquence initsync à l'aide de la CLI hermes-server, comme expliqué dans la section [Exécution](/setup/run/)

5. Configurez un premier client hermes en suivant les sections suivantes :
    - [hermes](/setup/hermes/) pour les paramètres globaux
    - configurez votre [plugin consommateur de bus de messages](/setup/configuration/plugins/messagebus_consumers/)
    - configurez vos [plugins d'attribut](/setup/configuration/plugins/attributes/), le cas échéant
    - configurez votre [plugin hermes-client](/setup/configuration/plugins/hermes-client/)
    - [hermes-client](/setup/configuration/hermes-client/) pour les paramètres génériques hermes-client et configurez votre modèle de données client

6. Exécutez le client hermes approprié en suivant la section [Exécution](/setup/run/)
