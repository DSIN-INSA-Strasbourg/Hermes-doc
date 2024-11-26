---
title: Concepts clé
weight: 3
---

## Source de données {#datasource}

Une source depuis laquelle le [serveur](#server) va collecter des données. Cela peut être tout ce qui contient des données : base de données, annuaire LDAP, service Web, fichier plat...

## Plugin source de données {#datasource-plugin}

Un plugin Hermes chargé de collecter des données à partir d'un type de [source de données](#datasource) spécifique et de les fournir au [serveur](#server).

## Serveur {#server}

application hermes-server : interroge les sources de données à intervalles réguliers et convertit tous les changements entre les données récentes et les données précédentes en événements qui seront envoyés sur un [bus de messages](#message-bus) par un [plugin producteur de bus de messages](#message-bus-producer-plugin).

## Bus de messages {#message-bus}

Service externe comme [Apache Kafka](https://kafka.apache.org/) ou [RabbitMQ](https://www.rabbitmq.com/) qui collecte les événements du [serveur](#server) et les fournit aux [clients](#client) dans le même ordre que celui où ils ont été émis.

## Plugin producteur de bus de messages {#message-bus-producer-plugin}

Un plugin Hermes exécuté par le [serveur](#server), chargé d'émettre des événements sur un type de [bus de messages](#message-bus) spécifique.

## Plugin consommateur de bus de messages {#message-bus-consumer-plugin}

Un plugin Hermes exécuté par les [clients](#client), chargé de consommer des événements depuis un type de [bus de messages](#message-bus) spécifique.

## Client

application hermes-client : consomme les événements du [bus de messages](#message-bus) via le [plugin consommateur de bus de messages](#message-bus-consumer-plugin) et appelle les méthodes appropriées implémentées sur le [plugin client](#client-plugin) pour propager les changements de données sur la cible.

### Corbeille {#trashbin}

Si activée dans la configuration, le client ne supprimera pas immédiatement les données mais les stockera dans la corbeille pendant le nombre de jours configuré. Si les données sont ajoutées à nouveau durant ce délai, le client les restaurera depuis la corbeille. Sinon, une fois la limite de rétention de la corbeille atteinte, les données seront supprimées.

Selon l'implémentation choisie sur le [plugin client](#client-plugin), cela peut permettre de nombreux scénarios, *e.g.* désactiver un compte ou le garder actif pendant une période de grâce.

### File d'erreurs {#error-queue}

Lorsqu'une exception est levée lors du traitement d'un événement sur le [plugin client](#client-plugin), l'événement est stocké dans une file d'erreurs. Tous les événements suivants concernant les mêmes objets de données ne seront pas traités mais stockés dans la file d'erreurs jusqu'à ce que le premier soit traité avec succès. Le traitement des événements dans la file d'attente d'erreurs est relancé périodiquement.

### Auto remédiation {#auto-remediation}

Parfois, un événement peut être stocké dans la [file d'erreurs](#error-queue) en raison d'un problème de données (*e.g.* un nom de groupe avec un point terminal génèrera une erreur sur Active Directory). Si le point terminal est ensuite supprimé du nom de groupe sur la [source de données](#datasource), l'événement `modified` sera stocké dans la [file d'erreurs](#error-queue) et ne sera pas traité tant que le précédent n'aura pas été traité, ce qui ne pourra jamais se produire sans procéder à une opération risquée et non-souhaitable : l'édition manuelle du fichier cache du [client](#client).

L'auto remédiation résout ce type de problèmes en fusionnant les événements d'un même objet dans la [file d'erreurs](#error-queue).
Elle n'est pas activée par défaut, car elle peut perturber l'ordre de traitement normal des événements.

## Plugin client {#client-plugin}

Un plugin Hermes exécuté par le [client](#client), chargé d'implémenter des méthodes simples de traitement d'événements visant à propager les changements de données vers un type de cible spécifique.

## Plugin d'attribut {#attribute-plugin}

Un plugin Hermes exécuté par le [serveur](#server) ou le [client](#client) qui sera utilisable comme un nouveau [filtre Jinja](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters), permettant de modifier des données.

## Initsync

Un [client](#client) ne peut pas commencer à traiter de nouveaux événements en toute sécurité sans disposer de l'ensemble du jeu de données complet préalable. Ainsi, le [serveur](#server) est capable d'envoyer une séquence d'événements spécifique appelée `initsync` qui contiendra le [modèle de données serveur](#server-datamodel) et l'ensemble de données. Un client déjà initialisé l'ignorera silencieusement, mais un client non initialisé le traitera pour initialiser sa cible en ajoutant toutes les entrées fournies par la séquence initsync, puis commencera ensuite à traiter normalement les événements suivants.

## Modèle de données (*datamodel*) {#datamodel}

Comme il existe des différences entre eux, veuillez consulter [modèle de données serveur](#server-datamodel) et [modèle de données client](#client-datamodel).

### Type de données {#data-type}

Également nommé "*type d'objet*". Un type de données avec son [mapping d'attributs](#attributes-mapping) qui sera géré par Hermes.

#### Clé primaire {#primary-key}

L'attribut [type de données](#data-type) qui permet de distinguer une entrée d'une autre. Il doit évidemment être unique.

### Modèle de données serveur {#server-datamodel}

Configuration des [types de données](#data-type) que le [serveur](#server) doit gérer, avec leurs [mapping d'attributs](#attributes-mapping) respectifs. Le nom de l'attribut distant est le nom de l'attribut utilisé sur la [source de données](#datasource).

Le modèle de données du serveur est construit en spécifiant les éléments suivants :

- Chaque [type de données](#data-type) avec :
  - sa [clé primaire](#primary-key)
  - ses [clés étrangères](#foreign-keys)
  - ses [contraintes d'intégrité](#integrity-constraints)
  - sa [politique de conflit de fusion](#merge-conflict-policy)
  - les noms et opérations de chacune de ses [sources de données](#datasource) avec :
    - son [mapping d'attributs](#attributes-mapping)
    - sa liste d'attributs spéciaux : [attributs locaux](#local-attributes), [attributs secrets](#secrets-attributes) et [attributs à ne mettre qu'en cache](#cache-only-attributes)
    - ses [contraintes de fusion](#merge-constraints)

#### Politique de conflit de fusion {#merge-conflict-policy}

Définit le comportement si un même attribut obtient des valeurs différentes sur différentes [sources de données](#datasource).

#### Contraintes de fusion {#merge-constraints}

Permet de déclarer des contraintes pour garantir la cohérence des données lors de la fusion des données, lorsque le [serveur](#server) interroge plusieurs [sources de données](#datasource).

#### Clés étrangères {#foreign-keys}

Permet de déclarer des clés étrangères dans un [type de données](#data-type), que les clients utiliseront pour appliquer leur politique de clés étrangères. Voir [Clés étrangères](/hermes/how-it-works/hermes-client/foreign-keys/) pour plus de détails.

#### Contraintes d'intégrité {#integrity-constraints}

Permet de déclarer des contraintes entre plusieurs [types de données](#data-type) pour assurer la cohérence des données.

##### Attributs à ne mettre qu'en cache {#cache-only-attributes}

Attributs du [modèle de données](#server-datamodel) qui seront uniquement stockés dans le cache, mais ne seront ni envoyés dans les événements, ni utilisés lors de la comparaison avec le cache.

##### Attributs secrets {#secrets-attributes}

Attributs du [modèle de données](#server-datamodel) qui contiendront des données sensibles, comme des mots de passe, et ne doivent jamais être stockés dans le cache ni affichés dans les journaux. Ils seront tout de même envoyés aux clients, à moins qu'ils ne soient également définis comme [attributs locaux](#local-attributes).

{{% notice style="note" %}}
Comme ces attributs ne sont pas mis en cache, ils seront considérés comme ajoutés CHAQUE fois que le [serveur](#server) interrogera les sources de données.
{{% /notice %}}

##### Attributs locaux {#local-attributes}

Attributs du [modèle de données](#server-datamodel) qui ne seront pas envoyés dans les événements, mis en cache ni utilisés lors de la comparaison avec le cache, mais qui pourront être utilisés dans les templates Jinja.

### Modèle de données client {#client-datamodel}

Configuration des [types de données](#data-type) que le [client](#client) doit gérer, avec leur [mapping d'attributs](#attributes-mapping). Le nom de l'attribut distant est le nom de l'attribut utilisé sur le [modèle de données du serveur](#server-datamodel).

{{% notice style="info" %}}
Si vous vous demandez pourquoi ce mapping est nécessaire, voici pourquoi :

1. il permet la transformation des données locales via des [filtres Jinja](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters) et les [plugins d'attribut](#attribute-plugin) sur le [client](#client).
2. il permet de réutiliser (et de partager) les [plugins client](#client-plugin) sans nécessiter de modification du [modèle de données serveur](#server-datamodel) ni du code du plugin, mais simplement en modifiant le fichier de configuration [client](#client).

{{% /notice %}}

Le modèle de données client est construit en spécifiant les éléments suivants :

- Chaque [type de données](#data-type) avec :
  - le nom de son [type de données](#data-type) distant correspondant, appelé `hermesType`
  - son [mapping d'attributs](#attributes-mapping)

### Mapping d'attributs {#attributes-mapping}

Également appelé "*attrsmapping*". Mapping (clé/valeur) qui associe le nom de l'attribut interne (en tant que clé) à celui distant (en tant que valeur). Le distant peut être un modèle Jinja pour transformer des données avec des [filtres Jinja](https://jinja.palletsprojects.com/en/3.1.x/templates/#filters) et des [plugins d'attribut](#attribute-plugin).
