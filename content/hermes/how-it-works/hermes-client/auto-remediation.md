---
title: Auto remediation
weight: 3
---

Sometimes, an event may be stored in error queue due to a data problem (*e.g.* a group name with a trailing dot will raise an error on Active Directory). If the trailing dot is then removed from the group name on datasource, the *modified* event will be stored on error queue, and won’t be processed until previous one is processed, which cannot happen without proceeding to a risky and undesirable operation: manually editing client cache file.

The autoremediation solves this type of problems by merging events of a same object in error queue. It is not enabled by default, as it may break the regular processing order of events.

## Example

Let's take an example with a group created with an invalid name. As its name is invalid, its processing will fail, and the event will be stored in error queue like this:

``` mermaid
flowchart TB
  subgraph errorqueue [Error queue]
    direction TB
    ev1
  end

  ev1["`**event 1**
    &nbsp;
    *eventtype*: added
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;grp_pkey: 42
    &nbsp;&nbsp;name: 'InvalidName.'
    &nbsp;&nbsp;desc: 'Demo group'
    }`"]

  classDef leftalign text-align:left
  class ev1 leftalign
```

As the error has been notified, someone corrects the group name in the datasource. This change will conduce to an according *modified* event. This *modified* event will not be processed, but added to the error queue as its object already has an event in error queue.

- without autoremediation, until the first event has been successfully processed, the second one will not even be tried. The fix is stuck.
- with autoremediation, the error queue will merge the two events, and then on the next processing of error queue, the updated event will be successfully processed.

``` mermaid
flowchart TB
  subgraph errorqueuebis [Error queue with autoremediation]
    direction TB
    ev1bis
  end

  subgraph errorqueue [Error queue without autoremediation]
    direction TB
    ev1
    ev2
  end

  ev1["`**event 1**
    &nbsp;
    *eventtype*: added
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;grp_pkey: 42
    &nbsp;&nbsp;name: 'InvalidName.'
    &nbsp;&nbsp;desc: 'Demo group'
    }`"]

  ev2["`**event 2**
    &nbsp;
    *eventtype*: modified
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;modified: {
    &nbsp;&nbsp;&nbsp;&nbsp;name: 'ValidName'
    &nbsp;&nbsp;}
    }`"]

  ev1bis["`**event 1**
    &nbsp;
    *eventtype*: added
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;grp_pkey: 42
    &nbsp;&nbsp;name: 'ValidName'
    &nbsp;&nbsp;desc: 'Demo group'
    }`"]

  classDef leftalign text-align:left
  class ev1,ev2,ev1bis leftalign
```
