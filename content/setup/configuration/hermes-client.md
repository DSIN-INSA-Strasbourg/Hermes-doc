---
title: hermes-client
weight: 3
---

Settings shared by all clients.

Main subsections :

- hermes-client
  - [datamodel](#hermes-client.datamodel)
    - *data-type-name*
      - [attrsmapping](#hermes-client.datamodel.data-type-name.attrsmapping)

---

## hermes-client.enableAutoremediation {#hermes-client.enableAutoremediation}

- *Description* : If `true`, will try to remediate errors by merging `added` and `modified` events data in error queue when possible.
  {{% notice warning %}}
  Enabling this feature will break the processing order of events : if your data types are only linked by primary keys, it shouldn't be problematic, but if the links between them are more complex, you really should consider what could go wrong before enabling it.  

  See [how works autoremediation](../../how-it-works/hermes-client/#auto-remediation) for more details.
  {{% /notice%}}
- *Mandatory* : No
- *Type* : boolean
- *Default value* : `false`

## hermes-client.errorQueue_retryInterval {#hermes-client.errorQueue_retryInterval}

- *Description* : Number of minutes between two attempts of re-processing events in error.
- *Mandatory* : No
- *Type* : integer
- *Default value* : 60 *(1 hour)*
- *Valid values* : 1 - 65535

## hermes-client.trashbin_purgeInterval {#hermes-client.trashbin_purgeInterval}

- *Description* : Number of minutes between two trashbin purge attempts.  
  *Ignored if [trashbin_retention](#hermes-client.trashbin_retention) is `0`/`unset`*.
- *Mandatory* : No
- *Type* : integer
- *Default value* : 60 *(1 hour)*
- *Valid values* : 1 - 65535

## hermes-client.trashbin_retention {#hermes-client.trashbin_retention}

- *Description* : Number of days to keep removed data in trashbin before permanently deleting it.  
  `0`/`unset` disable the trashbin : data will be immediatly deleted.
- *Mandatory* : No
- *Type* : integer
- *Default value* : 0 *(no trashbin)*
- *Valid values* : >= 0

## hermes-client.updateInterval {#hermes-client.updateInterval}

- *Description* : Number of seconds to sleep once no more events are available on message bus.
- *Mandatory* : No
- *Type* : integer
- *Default value* : 5
- *Valid values* : >= 0

## hermes-client.useFirstInitsyncSequence {#hermes-client.useFirstInitsyncSequence}

- *Description* : If true, indicate to use the first/older initsync sequence available on message bus. If false, the latest/newer will be used.
- *Mandatory* : No
- *Type* : boolean
- *Default value* : `false`

## hermes-client.datamodel {#hermes-client.datamodel}

Mandatory subsection used to configure [client datamodel](/en/hermes/key-concepts/#client-datamodel).

For each [data types](/en/hermes/key-concepts/#data-type) needed, a subsection with the desired data type name must be created, and configured. The data type name MUST start with an alphanumerical character.

Obviously, at least one data type must be set up.

### hermes-client.datamodel.data-type-name.hermesType {#hermes-client.datamodel.data-type-name.hermesType}

- *Description* : Name of corresponding data type on `hermes-server`.
- *Mandatory* : **Yes**
- *Type* : string

### hermes-client.datamodel.data-type-name.toString {#hermes-client.datamodel.data-type-name.toString}

- *Description* : Jinja template to compose the way an data item will be represented in log files.
- *Mandatory* : No
- *Type* : string

### hermes-client.datamodel.data-type-name.attrsmapping {#hermes-client.datamodel.data-type-name.attrsmapping}

Mandatory subsection to set up attribute mapping. *CLIENT* attributes as keys, *REMOTE* attributes (identified as *HERMES* attributes on hermes-server) as values.

A Jinja template could be set as value. If you do so, the value outside the templates will be used as raw string, and not as remote attribute name.

Jinja vars available are :

- each remote attribute for current data type, only if its value is not `NULL` and not an empty list.
