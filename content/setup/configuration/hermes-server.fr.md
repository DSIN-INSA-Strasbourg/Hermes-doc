---
title: hermes-server
weight: 2
---

Paramètres du serveur.

Sous-sections principales :

- [hermes-server](/setup/configuration/hermes-server/#hermes-server)
  - [datamodel](/setup/configuration/hermes-server/#hermes-server.datamodel)
    - *data-type-name*
      - [sources](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources)
        - *datasource-name*
          - [fetch](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch)
            - [vars](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars)
          - [commit_one](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one)
            - [vars](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars)
          - [commit_all](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all)
            - [vars](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars)
          - [attrsmapping](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping)

---

## hermes-server {#hermes-server}

### hermes-server.updateInterval {#hermes-server.updateInterval}

- *Description* : Intervalle entre deux actualisations des données, en secondes.
- *Obligatoire* : **Oui**
- *Type* : integer
- *Valeurs valides* : >= 0

### hermes-server.datamodel {#hermes-server.datamodel}

Sous-section obligatoire utilisée pour configurer le [modèle de données serveur](/hermes/hermes/key-concepts/#server-datamodel).

Pour chaque [type de données](/hermes/hermes/key-concepts/#data-type) requis, une sous-section avec le nom souhaité du type de données doit être créée et configurée. Le nom du type de données DOIT commencer par un caractère alphanumérique.

Bien évidemment, au moins un type de données doit être configuré.

{{% notice note %}}
L'ordre de déclaration des types de données est important pour garantir l'intégrité des données :

- les événements d'ajout/modification seront traités dans l'ordre de déclaration
- les événements de suppression seront traités dans l'ordre de déclaration inversé

Vous devez donc d'abord déclarer les types de données qui ne dépendent d'aucun autre type, puis les types qui ont des dépendances (clés étrangères) à ceux déclarés au dessus.

{{% /notice %}}

#### hermes-server.datamodel.data-type-name.primarykeyattr {#hermes-server.datamodel.data-type-name.primarykeyattr}

- *Description* : Le nom de l'attribut du modèle de données utilisé comme [clé primaire](/hermes/hermes/key-concepts/#primary-key). Si la clé primaire est un tuple, vous pouvez déclarer une liste de noms.
- *Obligatoire* : **Oui**
- *Type* : string | string[]

#### hermes-server.datamodel.data-type-name.toString {#hermes-server.datamodel.data-type-name.toString}

- *Description* : Template Jinja permettant de définir la représentation d'une entrée dans les fichiers journaux.
- *Obligatoire* : Non
- *Type* : string

#### hermes-server.datamodel.data-type-name.on_merge_conflict {#hermes-server.datamodel.data-type-name.on_merge_conflict}

- *Description* : Définit le comportement si un même attribut a une valeur différente sur plusieurs sources.
- *Obligatoire* : Non
- *Type* : string
- *Valeur par défaut* : use_cached_entry
- *Valeurs valides* :
  - `keep_first_value`: utilise la première valeur rencontrée dans l'ordre de déclaration des sources.
  - `use_cached_entry`: ignore les données récupérées et continue à utiliser l'entrée en cache jusqu'à ce que le conflit soit résolu.

#### hermes-server.datamodel.data-type-name.foreignkeys {#hermes-server.datamodel.data-type-name.foreignkeys}

- *Description* : Permet de déclarer des clés étrangères dans un type de données, que les clients utiliseront pour appliquer leur politique de clés étrangères. Voir [Clés étrangères](/hermes/hermes/how-it-works/hermes-client/foreign-keys/) pour plus de détails.  
Le paramètre est un dictionnaire avec la clé primaire du type de données actuel comme clé, un dictionnaire avec deux entrées comme valeur, faisant référence au type de données parent `from_objtype` et à sa clé primaire `from_attr`.  
Bien que cela puisse sembler intuitif, **la déclaration de clés étrangères ne créera aucune règle de contrainte d'intégrité automatiquement.**
{{% notice warning %}}
Que ce soit pour le type de données courant ou pour le parent, les attributs doivent être des clés primaires de leurs types respectifs.  
De plus, la clé primaire du parent ne peut pas être multivaluée (un tuple).

Ces contraintes pourraient éventuellement être assouplies un jour, mais pour l'instant aucun cas d'utilisation pertinent n'en justifie le besoin.
{{% /notice %}}

  Exemple :

  ```yaml
  foreignkeys:
    group_id:
      from_objtype: SRVGroups
      from_attr: gid
    user_id:
      from_objtype: SRVUsers
      from_attr: uid
  ```

- *Obligatoire* : Non
- *Type* : dict[string, dict[string, string]]
- *Valeur par défaut* : {}

#### hermes-server.datamodel.data-type-name.integrity_constraints {#hermes-server.datamodel.data-type-name.integrity_constraints}

- *Description* : Contraintes d'intégrité entre les types de données, en Jinja.  
  ATTENTION : cela peut être affreusement lent, vous devriez donc essayer de faire au plus simple et idéalement vous limiter aux clés primaires.

  Les variables Jinja disponibles sont :
  - **_SELF** : l'objet actuel
  - ***data-type-name*_pkeys** : un ensemble (*set*) contenant chaque clé primaire du type de données spécifié.
  - ***data-type-name*** : une liste de dictionnaires contenant chaque entrée du type de données spécifié.

  Exemple :

  ```yaml
  integrity_constraints:
    - "{{ _SELF.pkey_attr in OTHERDataType_pkeys }}"
  ```

- *Obligatoire* : Non
- *Type* : string[]
- *Valeur par défaut* : []

#### hermes-server.datamodel.data-type-name.sources {#hermes-server.datamodel.data-type-name.sources}

Sous-section obligatoire listant les sources de données à utiliser pour récupérer les données du type de données courant.

Pour chaque source de données utilisée, une sous-section portant son nom doit être définie et configurée.

Évidemment, au moins une source de données doit être configurée.

{{% notice note %}}
L'ordre de déclaration des sources de données est important pour la fusion des données si [hermes-server.datamodel.data-type-name.on_merge_conflict](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.on_merge_conflict) est défini sur `keep_first_value`, ou si [hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint) est utilisé.
{{% /notice %}}

##### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch}

Sous-section obligatoire pour configurer la requête utilisée pour récupérer les données.

Selon le plugin de source de données utilisé, [query](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.query) et [vars](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars) peuvent être facultatifs : configurez-les selon la [documentation du plugin de source de données](/setup/configuration/plugins/datasources/).

###### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.type {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.type}

- *Description* : Indique au plugin de source de données le type de requête à exécuter. Il s'agit généralement de `fetch`.
- *Obligatoire* : **Oui**
- *Type* : string
- *Valeurs valides* :
  - `fetch`: Indique que le plugin doit récupérer des données, sans modifier le jeu de données.
  - `add`: Indique que le plugin doit ajouter des données au jeu de données.
  - `delete`: Indique que le plugin doit supprimer des données du jeu de données.
  - `modify`: Indique que le plugin doit modifier des données dans le jeu de données.

###### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.query {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.query}

- *Description* : La requête à envoyer à la source de données. Peut être un template Jinja.

  Les variables Jinja disponibles sont :
  - **REMOTE_ATTRIBUTES** : la liste des noms d'attributs distants utilisés dans `attrsmapping`. Peut être utile pour générer des requêtes SQL avec les données requises sans utiliser de wildcards ni saisir manuellement la liste d'attributs.
  - **CACHED_VALUES** : le cache de la requête précédente. Une liste de dictionnaires dans laquelle chaque dictionnaire est une entrée avec le nom d'attribut comme clé et sa valeur comme valeur. Peut être utile pour filtrer la requête à l'aide d'une valeur mise en cache.
  - ***data-type-name*_pkeys** : un ensemble (*set*) contenant chaque clé primaire du type de données spécifié. Le type de données de la variable doit être déclaré avant celui actuel dans le modèle de données, sinon le contenu de la variable sera toujours vide car son contenu sera récupéré après celui du type de données actuel.
  - ***data-type-name*** : une liste de dictionnaires contenant chaque entrée du type de données spécifié. Le type de données de la variable doit être déclaré avant celui courant dans le modèle de données, sinon le contenu de la variable sera toujours vide car son contenu sera récupéré après celui du type de données courant.
- *Obligatoire* : Non
- *Type* : string

###### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars}

Sous-section facultative contenant certaines variables à transmettre au plugin de source de données.

Le nom de la variable comme clé et sa valeur comme valeur. Chaque valeur peut être un template Jinja.

Les variables Jinja disponibles sont :

- **REMOTE_ATTRIBUTES** : la liste des noms d'attributs distants utilisés dans `attrsmapping`. Peut être utile pour générer des requêtes SQL avec les données requises sans utiliser de wildcards ni saisir manuellement la liste d'attributs.
- **CACHED_VALUES** : le cache de la requête précédente. Une liste de dictionnaires dans laquelle chaque dictionnaire est une entrée avec le nom d'attribut comme clé et sa valeur comme valeur.
- ***data-type-name*_pkeys** : un ensemble (*set*) contenant chaque clé primaire du type de données spécifié. Le type de données de la variable doit être déclaré avant celui actuel dans le modèle de données, sinon le contenu de la variable sera toujours vide car son contenu sera récupéré après celui du type de données actuel.
- ***data-type-name*** : une liste de dictionnaires contenant chaque entrée du type de données spécifié. Le type de données de la variable doit être déclaré avant celui courant dans le modèle de données, sinon le contenu de la variable sera toujours vide car son contenu sera récupéré après celui du type de données courant.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one}

Sous-section facultative permettant de configurer une requête à exécuter à chaque fois qu'une entrée a été traitée sans erreur.

Selon le plugin de source de données utilisé, [query](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.query) et [vars](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.vars) peuvent être facultatifs : configurez-les selon la [documentation du plugin de source de données](/setup/configuration/plugins/datasources/).

{{% notice warning %}}
[commit_one](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one) et [commit_all](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all) s'excluent mutuellement : vous pouvez définir aucun ou l'un d'entre eux, mais pas les deux en même temps.
{{% /notice %}}

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.type {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.type}

- *Description* : Indique au plugin de source de données le type de requête à exécuter.
- *Obligatoire* : **Oui**
- *Type* : string
- *Valeurs valides* :
  - `fetch`: Indique que le plugin doit récupérer des données, sans modifier le jeu de données.
  - `add`: Indique que le plugin doit ajouter des données au jeu de données.
  - `delete`: Indique que le plugin doit supprimer des données du jeu de données.
  - `modify`: Indique que le plugin doit modifier des données dans le jeu de données.

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.query {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.query}

- *Description* : La requête à envoyer à la source de données. Peut être un template Jinja.

  Les variables Jinja disponibles sont :
  - **REMOTE_ATTRIBUTES** : la liste des noms d'attributs distants utilisés dans `attrsmapping`. Peut être utile pour générer des requêtes SQL avec les données requises sans utiliser de wildcards ni saisir manuellement la liste d'attributs.
  - **ITEM_CACHED_VALUES** : la valeur de l'entrée courante en cache. Un dictionnaire avec le nom d'attribut comme clé et sa valeur comme valeur.
  - **ITEM_FETCHED_VALUES** : la valeur de l'entrée courante fraîchement récupérée. Un dictionnaire avec le nom d'attribut comme clé et sa valeur comme valeur.
- *Obligatoire* : Non
- *Type* : string

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.vars {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.vars}

Sous-section facultative contenant certaines variables à transmettre au plugin de source de données.

Le nom de la variable comme clé et sa valeur comme valeur. Chaque valeur peut être un template Jinja.

Les variables Jinja disponibles sont :

- **REMOTE_ATTRIBUTES** : la liste des noms d'attributs distants utilisés dans `attrsmapping`. Peut être utile pour générer des requêtes SQL avec les données requises sans utiliser de wildcards ni saisir manuellement la liste d'attributs.
- **ITEM_CACHED_VALUES** : la valeur de l'entrée courante en cache. Un dictionnaire avec le nom d'attribut comme clé et sa valeur comme valeur.
- **ITEM_FETCHED_VALUES** : la valeur de l'entrée courante fraîchement récupérée. Un dictionnaire avec le nom d'attribut comme clé et sa valeur comme valeur.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all}

Sous-section facultative permettant de configurer une requête à exécuter une fois que toutes les données ont été traitées sans erreur.

Selon le plugin de source de données utilisé, [query](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.query) et [vars](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.vars) peuvent être facultatifs : configurez-les selon la [documentation du plugin de source de données](/setup/configuration/plugins/datasources/).

{{% notice warning %}}
[commit_all](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all) et [commit_one](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one) s'excluent mutuellement : vous pouvez définir aucun ou l'un d'entre eux, mais pas les deux en même temps.
{{% /notice %}}

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.type {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.type}

- *Description* : Indique au plugin de source de données le type de requête à exécuter.
- *Obligatoire* : **Oui**
- *Type* : string
- *Valeurs valides* :
  - `fetch`: Indique que le plugin doit récupérer des données, sans modifier le jeu de données.
  - `add`: Indique que le plugin doit ajouter des données au jeu de données.
  - `delete`: Indique que le plugin doit supprimer des données du jeu de données.
  - `modify`: Indique que le plugin doit modifier des données dans le jeu de données.

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.query {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.query}

- *Description* : La requête à envoyer à la source de données. Peut être un template Jinja.

  Les variables Jinja disponibles sont :
  - **REMOTE_ATTRIBUTES** : la liste des noms d'attributs distants utilisés dans `attrsmapping`. Peut être utile pour générer des requêtes SQL avec les données requises sans utiliser de wildcards ni saisir manuellement la liste d'attributs.
  - **CACHED_VALUES** : le cache de la requête précédente. Une liste de dictionnaires dans laquelle chaque dictionnaire est une entrée avec le nom d'attribut comme clé et sa valeur comme valeur. Peut être utile pour filtrer la requête à l'aide d'une valeur mise en cache.
  - **FETCHED_VALUES**: les valeurs fraîchement récupérées. Une liste de dictionnaires dans laquelle chaque dictionnaire est une entrée avec le nom d'attribut comme clé et sa valeur comme valeur. Peut être utile pour filtrer la requête à l'aide d'une valeur mise en cache.
- *Obligatoire* : Non
- *Type* : string

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.vars {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.vars}

Sous-section facultative contenant certaines variables à transmettre au plugin de source de données.

Le nom de la variable comme clé et sa valeur comme valeur. Chaque valeur peut être un template Jinja.

Les variables Jinja disponibles sont :

- **REMOTE_ATTRIBUTES** : la liste des noms d'attributs distants utilisés dans `attrsmapping`. Peut être utile pour générer des requêtes SQL avec les données requises sans utiliser de wildcards ni saisir manuellement la liste d'attributs.
- **CACHED_VALUES** : le cache de la requête précédente. Une liste de dictionnaires dans laquelle chaque dictionnaire est une entrée avec le nom d'attribut comme clé et sa valeur comme valeur. Peut être utile pour filtrer la requête à l'aide d'une valeur mise en cache.
- **FETCHED_VALUES**: les valeurs fraîchement récupérées. Une liste de dictionnaires dans laquelle chaque dictionnaire est une entrée avec le nom d'attribut comme clé et sa valeur comme valeur. Peut être utile pour filtrer la requête à l'aide d'une valeur mise en cache.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping {#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping}

Sous-section obligatoire pour configurer le mapping d'attributs. Les attributs *HERMES* comme clés, les attributs *DISTANTS* (sur la source de données) comme valeurs.
Une liste de plusieurs attributs distants peut être définie pour plus de commodité, leurs valeurs non-`NULL` seront combinées dans une liste.
Les valeurs `NULL` et les listes vides ne seront pas chargées.

Un template Jinja peut être défini comme valeur. Si vous le faites, la valeur entière doit être un
template. Vous ne pouvez pas définir `"{{ ATTRIBUTE.split('separator') }} SOME_NON_JINJA_ATTR"`.
Ceci est indispensable pour permettre à l'application d'identifier les *REMOTE_ATTRIBUTES*

Les variables Jinja disponibles sont :

- chaque attribut distant pour le type de données courant et la source de données dont sa valeur extraite, uniquement si sa valeur n'est pas `NULL` et n'est pas une liste vide.
- **ITEM_CACHED_VALUES**: la valeur de l'entrée courante en cache. Un dictionnaire avec le nom d'attribut comme clé et sa valeur comme valeur.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs {#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs}

- *Description* : Définit les attributs qui contiendront des données sensibles, comme des mots de passe.  
  Ceci indique à Hermes de ne pas les mettre en cache. Les noms d'attribut définis ici doivent exister en tant que clés dans [attrsmapping](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping). Ils seront envoyés aux clients à moins qu'ils ne soient également définis dans [local_attrs](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.local_attrs). Comme ils ne sont pas mis en cache, ils seront considérés comme ajoutés CHAQUE FOIS que le serveur sera redémarré, et les événements consécutifs à cet ajout seront émis.
- *Obligatoire* : Non
- *Type* : string[]

##### hermes-server.datamodel.data-type-name.sources.datasource-name.cacheonly_attrs {#hermes-server.datamodel.data-type-name.sources.datasource-name.cacheonly_attrs}

- *Description* : Définit les attributs qui seront uniquement stockés dans le cache.  
  Ils ne seront pas envoyés dans les événements, ni utilisés pour calculer le différentiel avec le cache. Les noms d'attribut définis ici doivent exister en tant que clés dans [attrsmapping](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping).
- *Obligatoire* : Non
- *Type* : string[]

##### hermes-server.datamodel.data-type-name.sources.datasource-name.local_attrs {#hermes-server.datamodel.data-type-name.sources.datasource-name.local_attrs}

- *Description* : Définit les attributs qui ne seront pas envoyés aux clients, mis en cache ni utilisés pour calculer le différentiel avec le cache.  
  Ils ne seront pas envoyés dans les événements, ni utilisés pour calculer le différentiel avec le cache. Les noms d'attributs définis ici doivent exister en tant que clés dans [attrsmapping](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping).
- *Obligatoire* : Non
- *Type* : string[]

##### hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint {#hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint}

- *Description* : Contraintes sur les clés primaires lors de la fusion : elles seront appliquées lors de la fusion des sources de données.  
  Comme la fusion sera appliquée dans l'ordre de déclaration des sources de données dans le fichier de configuration, la première contrainte source sera ignorée (puisqu'elle sera créée et non fusionnée).
  Ensuite, les premières données sources seront fusionnées avec celles de la deuxième source conformément à la `pkey_merge_constraint` de la deuxième. Ensuite, les données résultantes seront fusionnées avec les données sources de la troisième source conformément à la `pkey_merge_constraint` de la troisième, etc.
- *Obligatoire* : Non
- *Type* : string
- *Valeur par défaut* : `noConstraint`
- *Valeurs valides* :
  - `noConstraint`: n'applique aucune contrainte de fusion
  - `mustNotExist`: la clé primaire de la source actuelle ne doit pas exister dans la précédente (dans l'ordre de déclaration des sources de données), sinon les données de la source actuelle seront supprimées
  - `mustAlreadyExist`: la clé primaire de la source actuelle doit déjà exister dans la précédente (dans l'ordre de déclaration des sources de données), sinon les données de la source actuelle seront supprimées
  - `mustExistInBoth`: la clé primaire de la source actuelle doit déjà exister dans la précédente (dans l'ordre de déclaration des sources de données), sinon les données des deux sources seront supprimées

##### hermes-server.datamodel.data-type-name.sources.datasource-name.merge_constraints {#hermes-server.datamodel.data-type-name.sources.datasource-name.merge_constraints}

- *Description* : Contraintes de fusion avancées avec des règles Jinja.
  {{% notice warning %}}
  Horriblement lent, évitez au maximum de les utiliser.
  {{% /notice %}}
  Jinja vars available are:
  - **_SELF**: the data type item in current datasource being currently merged.
  - For each datasource declared in current data type:
    - ***datasource-name*_pkeys**: un ensemble (*set*) contenant chaque clé primaire du type de données courant dans la source de données spécifiée.
    - ***datasource-name***: les valeurs fraîchement récupérées depuis la source de données spéifiée. Une liste de dictionnaires dans laquelle chaque dictionnaire est une entrée avec le nom d'attribut comme clé et sa valeur comme valeur.
  {{% notice note %}}
  si [pkey_merge_constraint](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint) est défini, il sera appliqué **avant** `merge_constraints`, et les variables Jinja contiendront les valeurs résultantes.
  {{% /notice %}}
- *Obligatoire* : Non
- *Type* : string[]
