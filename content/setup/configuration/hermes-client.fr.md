---
title: hermes-client
weight: 3
---

Paramètres partagés par tous les clients.

Sous-sections principales :

- hermes-client
  - [datamodel](/setup/configuration/hermes-client/#hermes-client.datamodel)
    - *data-type-name*
      - [attrsmapping](/setup/configuration/hermes-client/#hermes-client.datamodel.data-type-name.attrsmapping)

---

## hermes-client.autoremediation {#hermes-client.autoremediation}

- *Description* : Politique d'auto remédiation à appliquer dans la file d'erreurs pour les événements concernant un même objet.
  {{% notice warning %}}
  L'activation de cette fonctionnalité peut modifier l'ordre de traitement normal des événements : si vos types de données ne sont liés que par des clés primaires, cela ne devrait pas poser de problème, mais si les liens entre eux sont plus complexes, vous devriez vraiment réfléchir à ce qui pourrait mal se passer avant de l'activer.

  *e.g.* avec la politique `maximum` et la [corbeille](/hermes/key-concepts/#trashbin) activée, l'auto remédiation supprimera les deux événements lorsqu'un événement `added` sera suivi d'un événement `removed`. Sans erreur, l'objet aurait été créé puis stocké dans la corbeille, mais dans ce cas, il ne sera jamais créé.

  Voir [comment fonctionne l'auto remédiation](/hermes/how-it-works/hermes-client/auto-remediation/) pour plus de détails.
  {{% /notice %}}
- *Obligatoire* : Non
- *Type* : string
- *Valeur par défaut* : `disabled`
- *Valeurs valides* :
  - `disabled` : pas d'auto remédiation, les événements sont empilés tels quels (*par défaut*).
  - `conservative` : fusionne uniquement les événements `added` et `modified` entre eux.
    - fusionne un événement `added` avec un événement `modified` suivant.
    - fusionne deux événements `modified` successifs.
  - `maximum` : fusionne tous les événements qui peuvent être fusionnés.
    - fusionne un événement `added` avec un événement `modified` suivant.
    - fusionne deux événements `modified` successifs.
    - supprime les deux événements lorsqu'un événement `added` est suivi d'un événement `removed`.
    - fusionne un événement `removed` suivi d'un événement `added` dans un événement `modified`.
    - supprime un événement `modified` lorsqu'il est suivi d'un événement `removed`.

## hermes-client.foreignkeys_policy {#hermes-client.foreignkeys_policy}

- *Description* : Définit les types d'événements qui seront placés dans la file d'erreurs si l'objet
    qui les concerne est le parent (par clé étrangère) d'un objet déjà présent dans la file d'erreurs.
    Voir [Clés étrangères](/hermes/how-it-works/hermes-client/foreign-keys/) pour plus de détails.
- *Obligatoire* : Non
- *Type* : string
- *Valeur par défaut* : `on_remove_event`
- *Valeurs valides* :
  - `disabled`: Aucun événement, la politique est désactivée.
  - `on_remove_event`: Uniquement sur les événements *removed*.
  - `on_every_event`: Tous les types d'événements (*added*, *modified*, *removed*)

## hermes-client.errorQueue_retryInterval {#hermes-client.errorQueue_retryInterval}

- *Description* : Nombre de minutes entre deux tentatives de traitement des événements en erreur.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 60 *(1 heure)*
- *Valeurs valides* : 1 - 65535

## hermes-client.trashbin_purgeInterval {#hermes-client.trashbin_purgeInterval}

- *Description* : Nombre de minutes entre deux tentatives de purge de la corbeille..
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 60 *(1 heure)*
- *Valeurs valides* : 1 - 65535
- *Ignoré lorsque* : [trashbin_retention](/setup/configuration/hermes-client/#hermes-client.trashbin_retention) vaut `0`/`unset`

## hermes-client.trashbin_retention {#hermes-client.trashbin_retention}

- *Description* : Nombre de jours pendant lesquels les données supprimées doivent être conservées dans la corbeille avant de les supprimer définitivement.  
  `0`/`unset` désactive la corbeille : les données seront immédiatement supprimées.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 0 *(pas de corbeille)*
- *Valeurs valides* : >= 0

## hermes-client.updateInterval {#hermes-client.updateInterval}

- *Description* : Nombre de secondes pendant lesquelles attendre une fois qu'il n'y a plus d'événements disponibles sur le bus de messages.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 5
- *Valeurs valides* : >= 0

## hermes-client.useFirstInitsyncSequence {#hermes-client.useFirstInitsyncSequence}

- *Description* : Si `true`, le client utilisera la première (plus ancienne) séquence initsync disponible sur le bus de messages. Si `false`, la dernière (plus récente) sera utilisée.
- *Obligatoire* : Non
- *Type* : boolean
- *Valeur par défaut* : `false`

## hermes-client.datamodel {#hermes-client.datamodel}

Sous-section obligatoire utilisée pour configurer le [modèle de données client](/hermes/key-concepts/#client-datamodel).

Pour chaque [type de données](/hermes/key-concepts/#data-type) requis, une sous-section avec le nom souhaité du type de données doit être créée et configurée. Le nom du type de données DOIT commencer par un caractère alphanumérique.

Bien évidemment, au moins un type de données doit être configuré.

### hermes-client.datamodel.data-type-name.hermesType {#hermes-client.datamodel.data-type-name.hermesType}

- *Description* : Nom du type de données correspondant sur `hermes-server`.
- *Obligatoire* : **Oui**
- *Type* : string

### hermes-client.datamodel.data-type-name.toString {#hermes-client.datamodel.data-type-name.toString}

- *Description* : Template Jinja permettant de définir la représentation d'une entrée dans les fichiers journaux.
- *Obligatoire* : Non
- *Type* : string

### hermes-client.datamodel.data-type-name.attrsmapping {#hermes-client.datamodel.data-type-name.attrsmapping}

Sous-section pour configurer le mapping d'attributs. Les attributs *CLIENT* comme clés, les attributs *DISTANTS* (identifiés comme des attributs *HERMES* sur le serveur hermes) comme valeurs.

Un template Jinja peut être défini comme valeur. Dans ce cas, toute valeur en dehors du template sera utilisée comme chaîne brute et non comme un nom d'attribut distant.

Les variables Jinja disponibles sont :

- chaque attribut distant pour le type de données actuel, uniquement si sa valeur n'est pas `NULL` et n'est pas une liste vide.

{{% notice note %}}
Si vous n'utilisez pas leur valeur, il n'est pas nécessaire de déclarer un mapping pour les clés primaires. Pour certains types de données, vous pouvez omettre le mapping d'attributs, ce qui équivaut à définir un modèle de données vide : il ne contiendra donc que sa ou ses clés primaires.
{{% /notice %}}
