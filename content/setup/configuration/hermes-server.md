---
title: hermes-server
weight: 2
---

Server settings.

Main subsections:

- [hermes-server](#hermes-server)
  - [datamodel](#hermes-server.datamodel)
    - *data-type-name*
      - [sources](#hermes-server.datamodel.data-type-name.sources)
        - *datasource-name*
          - [fetch](#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch)
            - [vars](#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars)
          - [commit_one](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one)
            - [vars](#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars)
          - [commit_all](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all)
            - [vars](#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars)
          - [attrsmapping](#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping)

---

## hermes-server {#hermes-server}

### hermes-server.updateInterval {#hermes-server.updateInterval}

- *Description*: Interval between two data updates, in seconds.
- *Mandatory*: **Yes**
- *Type*: integer
- *Valid values*: >= 0

### hermes-server.datamodel {#hermes-server.datamodel}

Mandatory subsection used to configure [server datamodel](../../../hermes/key-concepts/#server-datamodel).

For each [data types](../../../hermes/key-concepts/#data-type) needed, a subsection with the desired data type name must be created and configured. The data type name MUST start with an alphanumerical character.

Obviously, at least one data type must be set up.

{{% notice note %}}
The declaration order of data types is important to enforce data integrity:

- add/modify events will be processed in the declaration order
- remove events will be processed in the reversed declaration order

So you really should first declare data types that do not depend on any other types, and then types that have dependencies (foreign keys) to those declared above.

{{% /notice %}}

#### hermes-server.datamodel.data-type-name.primarykeyattr {#hermes-server.datamodel.data-type-name.primarykeyattr}

- *Description*: The name of the datamodel attribute used as [primary key](../../../hermes/key-concepts/#primary-key). If the primary key is a tuple, you may declare a list of names.
- *Mandatory*: **Yes**
- *Type*: string | string[]

#### hermes-server.datamodel.data-type-name.toString {#hermes-server.datamodel.data-type-name.toString}

- *Description*: Jinja template to compose the way a data item will be represented in log files.
- *Mandatory*: No
- *Type*: string

#### hermes-server.datamodel.data-type-name.on_merge_conflict {#hermes-server.datamodel.data-type-name.on_merge_conflict}

- *Description*: Behavior if a same attribute has different value on multiple sources.
- *Mandatory*: No
- *Type*: string
- *Default value*: use_cached_entry
- *Valid values*:
  - `keep_first_value`: use the first value met in source order.
  - `use_cached_entry`: ignore data fetched and keep using cached entry until conflict is solved.

#### hermes-server.datamodel.data-type-name.integrity_constraints {#hermes-server.datamodel.data-type-name.integrity_constraints}

- *Description*: Integrity constraints between datamodel type, in Jinja.  
  WARNING: it could be terribly slow, so you should keep it as simple as possible, and focus upon primary keys.

  Jinja vars available are:
  - **_SELF**: the current object
  - ***data-type-name*_pkeys**: a set with every primary key of specified data type.
  - ***data-type-name***: a list of dict containing each entry of specified data type.

  Example:

  ```yaml
  integrity_constraints:
    - "{{ _SELF.pkey_attr in OTHERDataType_pkeys }}"
  ```

- *Mandatory*: No
- *Type*: string[]
- *Default value*: []

#### hermes-server.datamodel.data-type-name.sources {#hermes-server.datamodel.data-type-name.sources}

Mandatory subsection listing the datasource(s) used to fetch current data type data.

For each datasource used, a subsection with its name must be defined and configured.

Obviously, at least one datasource must be set up.

{{% notice note %}}
The declaration order of datasources is important to for data merging if [hermes-server.datamodel.data-type-name.on_merge_conflict](#hermes-server.datamodel.data-type-name.on_merge_conflict) is set to `keep_first_value`, or if [hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint](#hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint) is used.
{{% /notice %}}

##### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch}

Mandatory subsection to set up the query used to fetch data.

According to datasource plugin used, [query](#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.query) and [vars](#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars) may be facultative: configure them according to your [datasource plugin documentation](../plugins/datasources/).

###### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.type {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.type}

- *Description*: Indicate to datasource plugin which flavor of query to proceed. Should probably be `fetch` here.
- *Mandatory*: **Yes**
- *Type*: string
- *Valid values*:
  - `fetch`: Indicate that plugin must fetch data, without altering dataset.
  - `add`: Indicate that plugin will add data to dataset.
  - `delete`: Indicate that plugin will delete data from dataset.
  - `modify`: Indicate that plugin will modify data in dataset.

###### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.query {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.query}

- *Description*: The query to send to datasource. May be a Jinja template.

  Jinja vars available are:
  - **REMOTE_ATTRIBUTES**: The list of remote attribute names used in `attrsmapping`. May be useful to generate SQL queries with required data without using wildcards or manually typing the attribute list.
  - **CACHED_VALUES**: The cache of previous query. A list of dictionaries, each dictionary is an entry with attrname as key, and corresponding value as value. May be useful to filter the query using a cached value.
  - ***data-type-name*_pkeys**: a set with every primary key of specified data type. The var's datatype must be declared before the current one in the datamodel, otherwise the content of the var will always be empty as its content will be fetched after that of the current datatype.
  - ***data-type-name***: a list of dict containing each entry of specified data type. The var's datatype must be declared before the current one in the datamodel, otherwise the content of the var will always be empty as its content will be fetched after that of the current datatype.
- *Mandatory*: No
- *Type*: string

###### hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars {#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch.vars}

Facultative subsection containing some vars to pass to datasource plugin.

The var name as key, and its value as value. Each value may be a Jinja template.

Jinja vars available are:

- **REMOTE_ATTRIBUTES**: The list of remote attribute names used in `attrsmapping`. May be useful to generate SQL queries with required data without using wildcards or manually typing the attribute list.
- **CACHED_VALUES**: The cache of previous query. A list of dictionaries, each dictionary is an entry with attrname as key, and corresponding value as value.
- ***data-type-name*_pkeys**: a set with every primary key of specified data type. The var's datatype must be declared before the current one in the datamodel, otherwise the content of the var will always be empty as its content will be fetched after that of the current datatype.
- ***data-type-name***: a list of dict containing each entry of specified data type. The var's datatype must be declared before the current one in the datamodel, otherwise the content of the var will always be empty as its content will be fetched after that of the current datatype.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one}

Facultative subsection to set up a query to run each time an item of current data has been processed without errors.

According to datasource plugin used, [query](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.query) and [vars](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.vars) may be facultative: configure them according to your [datasource plugin documentation](../plugins/datasources/).

{{% notice warning %}}
[commit_one](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one) and [commit_all](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all) are mutually exclusive: you can set none or one of them, but not both at the same time.
{{% /notice %}}

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.type {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.type}

- *Description*: Indicate to datasource plugin which flavor of query to proceed.
- *Mandatory*: **Yes**
- *Type*: string
- *Valid values*:
  - `fetch`: Indicate that plugin must fetch data, without altering dataset.
  - `add`: Indicate that plugin will add data to dataset.
  - `delete`: Indicate that plugin will delete data from dataset.
  - `modify`: Indicate that plugin will modify data in dataset.

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.query {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.query}

- *Description*: The query to send to datasource. May be a Jinja template.

  Jinja vars available are:
  - **REMOTE_ATTRIBUTES**: The list of remote attribute names used in `attrsmapping`. May be useful to generate SQL queries with required data without using wildcards or manually typing the attribute list.
  - **ITEM_CACHED_VALUES**: The cache values of current item. A dictionary with attrname as key, and corresponding value as value.
  - **ITEM_FETCHED_VALUES**: The fetched values of current item. A dictionary with attrname as key, and corresponding value as value.
- *Mandatory*: No
- *Type*: string

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.vars {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one.vars}

Facultative subsection containing some vars to pass to datasource plugin.

The var name as key, and its value as value. Each value may be a Jinja template.

Jinja vars available are:

- **REMOTE_ATTRIBUTES**: The list of remote attribute names used in `attrsmapping`. May be useful to generate SQL queries with required data without using wildcards or manually typing the attribute list.
- **ITEM_CACHED_VALUES**: The cache values of current item. A dictionary with attrname as key, and corresponding value as value.
- **ITEM_FETCHED_VALUES**: The fetched values of current item. A dictionary with attrname as key, and corresponding value as value.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all}

Facultative subsection to set up a query to run once all data have been processed with no errors.

According to datasource plugin used, [query](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.query) and [vars](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.vars) may be facultative: configure them according to your [datasource plugin documentation](../plugins/datasources/).

{{% notice warning %}}
[commit_all](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all) and [commit_one](#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one) are mutually exclusive: you can set none or one of them, but not both at the same time.
{{% /notice %}}

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.type {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.type}

- *Description*: Indicate to datasource plugin which flavor of query to proceed.
- *Mandatory*: **Yes**
- *Type*: string
- *Valid values*:
  - `fetch`: Indicate that plugin must fetch data, without altering dataset.
  - `add`: Indicate that plugin will add data to dataset.
  - `delete`: Indicate that plugin will delete data from dataset.
  - `modify`: Indicate that plugin will modify data in dataset.

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.query {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.query}

- *Description*: The query to send to datasource. May be a Jinja template.

  Jinja vars available are:
  - **REMOTE_ATTRIBUTES**: The list of remote attribute names used in `attrsmapping`. May be useful to generate SQL queries with required data without using wildcards or manually typing the attribute list.
  - **CACHED_VALUES**: The cache of previous polling. A list of dictionaries, each dictionary is an entry with attrname as key, and corresponding value as value.
  - **FETCHED_VALUES**: The fetched entries of current polling. A list of dictionaries, each dictionary is an entry with attrname as key, and corresponding value as value.
- *Mandatory*: No
- *Type*: string

###### hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.vars {#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_all.vars}

Facultative subsection containing some vars to pass to datasource plugin.

The var name as key, and its value as value. Each value may be a Jinja template.

Jinja vars available are:

- **REMOTE_ATTRIBUTES**: The list of remote attribute names used in `attrsmapping`. May be useful to generate SQL queries with required data without using wildcards or manually typing the attribute list.
- **CACHED_VALUES**: The cache of previous polling. A list of dictionaries, each dictionary is an entry with attrname as key, and corresponding value as value.
- **FETCHED_VALUES**: The fetched entries of current polling. A list of dictionaries, each dictionary is an entry with attrname as key, and corresponding value as value.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping {#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping}

Mandatory subsection to set up attribute mapping. *HERMES* attributes as keys, *REMOTE* attributes (on datasource) as values.
A list of several remote attributes can be defined as a convenience, their non-`NULL` values will be combined in a list.
The `NULL` values and empty lists won't be loaded.

A Jinja template could be set as value. If you do so, the whole value must be a
template. You can't set `"{{ ATTRIBUTE.split('separator') }} SOME_NON_JINJA_ATTR"`.
This is required to allow the software to collect the *REMOTE_ATTRIBUTES*

Jinja vars available are:

- each remote attribute for current data type and datasource with its fetched value, only if its value is not `NULL` and not an empty list.
- **ITEM_CACHED_VALUES**: The cache values of current item. A dictionary with attrname as key, and corresponding value as value.

##### hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs {#hermes-server.datamodel.data-type-name.sources.datasource-name.secrets_attrs}

- *Description*: Define attributes that will contain sensitive data, like passwords.  
  It will indicate Hermes to not cache them. The attribute names set here must exist as keys in [attrsmapping](#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping). They'll be sent to clients unless they're
  defined in [local_attrs](#hermes-server.datamodel.data-type-name.sources.datasource-name.local_attrs) too. As they're not cached, they'll be seen as added EACH TIME the server will be restarted, and the consecutive events will be sent.
- *Mandatory*: No
- *Type*: string[]

##### hermes-server.datamodel.data-type-name.sources.datasource-name.cacheonly_attrs {#hermes-server.datamodel.data-type-name.sources.datasource-name.cacheonly_attrs}

- *Description*: Define attributes that will only be stored in cache.  
  They won't be sent in events, nor used to diff with cache. The attribute names set here must exist as keys in [attrsmapping](#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping).
- *Mandatory*: No
- *Type*: string[]

##### hermes-server.datamodel.data-type-name.sources.datasource-name.local_attrs {#hermes-server.datamodel.data-type-name.sources.datasource-name.local_attrs}

- *Description*: Define attributes that won't be sent to clients, cached or used to diff with cache.  
  They won't be sent in events, nor used to diff with cache. The attribute names set here must exist as keys in [attrsmapping](#hermes-server.datamodel.data-type-name.sources.datasource-name.attrsmapping).
- *Mandatory*: No
- *Type*: string[]

##### hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint {#hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint}

- *Description*: Constraints on primary keys during merge: will be applied during datasources merge.  
  As merging will be processed in the datamodel source declaration order in config file, the first source constraint will be ignored (because it will be created and not merged).
  Then the first source data will be merged with the second source according to the second's `pkey_merge_constraint`. Then the resulting data will be merged with the third source data according to the third's `pkey_merge_constraint`, etc.
- *Mandatory*: No
- *Type*: string
- *Default value*: `noConstraint`
- *Valid values*:
  - `noConstraint`: don't apply any merge constraint
  - `mustNotExist`: the primary key in current source must not exist in previous (in datasources declaration order), otherwise the data of current will be discarded
  - `mustAlreadyExist`: the primary key in current source must already exist in previous (in datasources declaration order), otherwise the data of current will be discarded
  - `mustExistInBoth`: the primary key in current source must already exist in previous (in datasources declaration order), otherwise the data of both sources will be discarded

##### hermes-server.datamodel.data-type-name.sources.datasource-name.merge_constraints {#hermes-server.datamodel.data-type-name.sources.datasource-name.merge_constraints}

- *Description*: Advanced merge constraints with Jinja rules.
  {{% notice warning %}}
  Terribly slow, avoid using them as much as possible.
  {{% /notice %}}
  Jinja vars available are:
  - **_SELF**: the data type item in current datasource being currently merged.
  - For each datasource declared in current data type:
    - ***datasource-name*_pkeys**: A set with every primary key of data type item in current datasource.
    - ***datasource-name***: The fetched entries of current polling. A list of dictionaries, each dictionary is an entry with attrname as key, and corresponding value as value.
  {{% notice note %}}
  if [pkey_merge_constraint](#hermes-server.datamodel.data-type-name.sources.datasource-name.pkey_merge_constraint) is defined, it will be enforced **before** `merge_constraints`, and Jinja vars will contains the resulting values.
  {{% /notice %}}
- *Mandatory*: No
- *Type*: string[]
