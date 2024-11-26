---
title: Mise à jour du modèle de données serveur
weight: 1
---

Un modèle de données n'est pas figé dans le temps, il peut évoluer et donc être mis à jour, que ce soit depuis le serveur ou sur un ou plusieurs clients.

A chaque modification du modèle de données sur le serveur, sa nouvelle version est propagée aux clients avec ses données "publiques" : chaque type de données est inclus, avec sa clé primaire, la liste de ses attributs et la liste de ses attributs secrets. Des événements consécutifs sont ensuite émis.

## Ajouter un attribut à un type de données existant

1. 👱 Ajouter l'attribut au [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Emission d'événements "modified" pour les entrées concernées, avec l'attribut ajouté et sa valeur
    - 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma. Traitement des événements "modified" entrants : comme l'attribut n'est pas encore déclaré dans leurs modèles de données, sa valeur est ignorée mais stockée dans le cache complet
2. 👱 Ajouter l'attribut aux [modèles de données clients](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "modified" depuis le cache complet

ou

1. 👱 Ajouter l'attribut aux [modèles de données clients](/hermes/key-concepts/#client-datamodel) afin qu'ils puissent le traiter lorsqu'il sera ajouté au [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer les clients : ⚠️ avertissement du modèle de données "*remote attributes don't exist in current Dataschema*"
2. 👱 Ajouter l'attribut au [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "modified" pour les entrées concernées, avec l'attribut ajouté et sa valeur
3. 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma. ✅ Plus d'avertissement de modèle de données. Traitement des événements "modified" entrants

## Supprimer un attribut d'un type de données

1. 👱 Supprimer l'attribut des [modèles de données clients](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "modified" consécutifs
2. 👱 Supprimer l'attribut du [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "modified" pour les entrées concernées, avec l'attribut supprimé. Ils seront ignorés par les clients
3. 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma

ou

1. 👱 Supprimer l'attribut du [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "modified" pour les entrées concernées, avec l'attribut supprimé
2. 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma. ⚠️ avertissement du modèle de données "*remote attributes don't exist in current Dataschema*". Traitement des événements "modified" entrants
3. 👱 Supprimer l'attribut des [modèles de données clients](/hermes/key-concepts/#client-datamodel), redémarrer les clients: ✅ Plus d'avertissement de modèle de données

## Modifier la valeur d'un attribut (en changeant son filtre Jinja, ou son attribut distant de la source de données)

1. 👱 Modifier l'attribut dans le [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'événements "modified" pour les entrées concernées, avec les nouvelles valeurs de l'attribut modifié
2. 💻 Traitement des événements "modified" entrants

## Ajouter un attribut existant d'un type de données à *[secrets_attrs](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)*

1. 👱 Modifier *[secrets_attrs](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)* dans le [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Purge de l'attribut dans le cache du serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "modified" pour les entrées concernées, avec l'attribut ajouté et ses valeurs
2. 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma, purge de l'attribut dans leur cache
    - 💻 Traitement des événements "modified" entrants

## Supprimer un attribut existant d'un type de données de *[secrets_attrs](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)*

1. 👱 Modifier *[secrets_attrs](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs)* in [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "modified" pour les entrées concernées, avec l'attribut ajouté et ses valeurs
2. 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma
    - 💻 Traitement des événements "modified" entrants

## Ajouter un nouveau type de données

1. 👱 Ajouter le type de données au [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "added" pour chaque entrée du type de données ajouté
    - 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma. Traitement des événements "added" entrants : comme le type de données n'est pas encore déclaré dans leur modèle de données, ses entrées sont ignorées mais stockées dans le cache complet
2. 👱 Ajouter le type de données aux [modèles de données client](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "added" depuis le cache complet

ou

1. 👱 Ajouter le type de données aux [modèles de données client](/hermes/key-concepts/#client-datamodel) afin qu'ils puissent le traiter lorsqu'il sera ajouté au [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer les clients : ⚠️ avertissement du modèle de données "*remote types don't exist in current Dataschema*"
2. 👱 Ajouter le type de données au [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Émission d'événements "added" pour chaque entrée du type de données ajouté
3. 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma. ✅ Plus d'avertissement de modèle de données. Traitement des événements "added" entrants

## Supprimer un type de données existant

1. 👱 Supprimer le type de données dans les [modèles de données client](/hermes/key-concepts/#client-datamodel), redémarrer les clients
    - 💻 Traitement des mises à jour du modèle de données local par les clients : génération et traitement des événements locaux "removed" consécutifs
    - 💻 Purge des fichiers de cache locaux du type de données supprimé
2. 👱 Supprimer le type de données dans le [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'événements "removed" pour chaque entrée du type de données supprimé
    - 💻 Purge des fichiers de cache du type de données supprimé
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
3. 💻 Traitement des événements "removed" entrants par les clients : tous sont ignorés
    - 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma
    - 💻 Purge des fichiers de cache distants du type de données supprimé

ou

1. 👱 Supprimer le type de données dans le [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Émission d'événements "removed" pour chaque entrée du type de données supprimé
    - 💻 Purge des fichiers de cache du type de données supprimé
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
2. 💻 Traitement des événements "removed" entrants par les clients
    - 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : mise à jour de leur schéma. ⚠️ avertissement du modèle de données "*remote types don't exist in current Dataschema*"
    - 💻 Purge des fichiers de cache distants du type de données supprimé
3. 👱 Supprimer le type de données dans les [modèles de données client](/hermes/key-concepts/#client-datamodel), redémarrer les clients: ✅ Plus d'avertissement de modèle de données
    - 💻 Purge des fichiers de cache locaux du type de données supprimé

## Modifier l'attribut de clé primaire d'un type de données

{{% notice style="warning" title="DANGER" %}}
Il s'agit de la mise à jour du modèle de données la plus risquée, car il peut y avoir des liens entre les types de données, utilisant des clés primaires comme clés étrangères.
Cela signifie que vous devrez mettre à jour tous les types de données en une fois, sans rien manquer.

**Vous devriez vraiment envisager d'effectuer cette mise à jour dans un environnement de test avant de la faire en production**, car si quelque chose échoue, vos clients pourraient être définitivement compromis.
{{% /notice %}}

### Prérequis

Les attributs à utiliser comme nouvelle clé primaire doivent déjà exister dans votre [modèle de données serveur](/hermes/key-concepts/#server-datamodel), et leurs valeurs doivent déjà avoir été propagées et exister dans le cache des clients.

{{% notice style="tip" title="La rétention de la corbeille peut retarder la mise à jour du modèle de données" %}}
La nouvelle clé primaire **DOIT** exister dans chaque entrée de son type de données avant la mise à jour du modèle de données. Si la corbeille est activée sur certains de vos clients, le nouvel attribut de clé primaire peut être absent des entrées qui s'y trouvent.

La manière la plus sûre de gérer cela est d'ajouter l'attribut à votre [modèle de données serveur](/hermes/key-concepts/#server-datamodel) et de retarder le changement de clé primaire d'au moins un jour + autant de jours que la valeur [trashbin_retention](/setup/configuration/hermes-client/#hermes-client.trashbin_retention) la plus élevée de tous vos clients.

Si vous ne gérez pas cela de cette manière, le client purgera toutes les entrées en corbeille qui ne contiennent pas la valeur du ou des nouveaux attributs de clé primaire, comme si le délai [trashbin_retention](/setup/configuration/hermes-client/#hermes-client.trashbin_retention) était échu.
{{% /notice %}}

### Mise à jour

1. 👱 Mettre à jour tous les types de données dans le [modèle de données serveur](/hermes/key-concepts/#server-datamodel), redémarrer le serveur
    - 💻 Mise à jour des clés primaires modifiées dans les fichiers cache du serveur
    - 💻 Émission d'un [événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par le serveur
    - 💻 Traitement de l'[événement dataschema](/hermes/how-it-works/hermes-server/events-emitted/) par les clients : purge des entrées en corbeille auxquelles il manque la nouvelle clé primaire, mise à jour de leurs schémas, mise à jour des clés primaires modifiées dans les fichiers cache et la file d'erreurs
