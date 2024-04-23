---
title: hermes-client
weight: 3
---

Settings shared by all clients.

Main subsections:

- hermes-client
  - [datamodel](#hermes-client.datamodel)
    - *data-type-name*
      - [attrsmapping](#hermes-client.datamodel.data-type-name.attrsmapping)

---

## hermes-client.autoremediation {#hermes-client.autoremediation}

- *Description*: Autoremediation policy to use in error queue for events concerning a same object.
  {{% notice warning %}}
  Enabling this feature may break the regular processing order of events: if your data types are only linked by primary keys, it shouldn't be problematic, but if the links between them are more complex, you really should consider what could go wrong before enabling it.  

  *e.g.* with `maximum` policy, and [trashbin](../../../hermes/key-concepts/#trashbin) enabled, the autoremediation will delete both events when an `added` event is followed by a `removed` event. Without error, the object would have been created and stored in trashbin, but in this case it won't even be created.

  See [how autoremediation works](../../../hermes/how-it-works/hermes-client/auto-remediation/) for more details.
  {{% /notice%}}
- *Mandatory*: No
- *Type*: string
- *Default value*: `disabled`
- *Valid values*:
  - `disabled`: no autoremediation, events are stacked as is (*default*).
  - `conservative`: only merge `added` and `modified` events between them.
    - merge an `added` event with a following `modified` event.
    - merge two successive `modified` events.
  - `maximum`: merge every events that can be merged.
    - merge an `added` event with a following `modified` event.
    - merge two successive `modified` events.
    - delete both events when an `added` event is followed by a `removed` event.
    - merge a `removed` event followed by an `added` event in a `modified` event.
    - delete a `modified` event when it is followed by a `removed` event.

## hermes-client.errorQueue_retryInterval {#hermes-client.errorQueue_retryInterval}

- *Description*: Number of minutes between two attempts of re-processing events in error.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 60 *(1 hour)*
- *Valid values*: 1 - 65535

## hermes-client.trashbin_purgeInterval {#hermes-client.trashbin_purgeInterval}

- *Description*: Number of minutes between two trashbin purge attempts.  
  *Ignored if [trashbin_retention](#hermes-client.trashbin_retention) is `0`/`unset`*.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 60 *(1 hour)*
- *Valid values*: 1 - 65535

## hermes-client.trashbin_retention {#hermes-client.trashbin_retention}

- *Description*: Number of days to keep removed data in trashbin before permanently deleting it.  
  `0`/`unset` disable the trashbin: data will be immediately deleted.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 0 *(no trashbin)*
- *Valid values*: >= 0

## hermes-client.updateInterval {#hermes-client.updateInterval}

- *Description*: Number of seconds to sleep once no more events are available on message bus.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 5
- *Valid values*: >= 0

## hermes-client.useFirstInitsyncSequence {#hermes-client.useFirstInitsyncSequence}

- *Description*: If true, indicate to use the first/older initsync sequence available on message bus. If false, the latest/newer will be used.
- *Mandatory*: No
- *Type*: boolean
- *Default value*: `false`

## hermes-client.datamodel {#hermes-client.datamodel}

Mandatory subsection used to configure [client datamodel](../../../hermes/key-concepts/#client-datamodel).

For each [data types](../../../hermes/key-concepts/#data-type) needed, a subsection with the desired data type name must be created and configured. The data type name MUST start with an alphanumerical character.

Obviously, at least one data type must be set up.

### hermes-client.datamodel.data-type-name.hermesType {#hermes-client.datamodel.data-type-name.hermesType}

- *Description*: Name of corresponding data type on `hermes-server`.
- *Mandatory*: **Yes**
- *Type*: string

### hermes-client.datamodel.data-type-name.toString {#hermes-client.datamodel.data-type-name.toString}

- *Description*: Jinja template to compose the way a data item will be represented in log files.
- *Mandatory*: No
- *Type*: string

### hermes-client.datamodel.data-type-name.attrsmapping {#hermes-client.datamodel.data-type-name.attrsmapping}

Subsection to set up attribute mapping. *CLIENT* attributes as keys, *REMOTE* attributes (identified as *HERMES* attributes on hermes-server) as values.

A Jinja template could be set as value. If you do so, the value outside the templates will be used as raw string, and not as remote attribute name.

Jinja vars available are:

- each remote attribute for current data type, only if its value is not `NULL` and not an empty list.

{{% notice note %}}
If you won't use their value, it is not necessary to declare a mapping for primary key(s). For some data types, you may omit the attrsmapping, which is equivalent to defining an empty one : therefore it will only contain its primary key(s).
{{% /notice %}}
