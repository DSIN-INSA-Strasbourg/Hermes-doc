---
title: Auto remédiation
weight: 4
---

Parfois, un événement peut être stocké dans la file d'erreurs en raison d'un problème de données (e.g. un nom de groupe avec un point terminal génère une erreur sur Active Directory). Si le point terminal est ensuite supprimé du nom de groupe sur la source de données, l'événement *modified* sera stocké dans la file d'erreurs et ne sera pas traité tant que le précédent n'aura pas été traité, ce qui ne pourra jamais se produire sans procéder à une opération risquée et non-souhaitable : l'édition manuelle du fichier cache du client.

L'auto remédiation résout ce type de problèmes en fusionnant les événements d'un même objet dans la file d'erreurs. Elle n'est pas activée par défaut, car elle peut perturber l'ordre de traitement normal des événements.

## Exemple

Prenons comme exemple un groupe créé avec un nom invalide. Comme son nom est invalide, le traitement de sa création échouera et l'événement sera stocké dans la file d'erreurs comme ceci :

``` mermaid
flowchart TB
  subgraph errorqueue [File d'erreurs]
    direction TB
    ev1
  end

  ev1["`**événement 1**
    &nbsp;
    *eventtype*: added
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;grp_pkey: 42
    &nbsp;&nbsp;name: 'NomInvalide.'
    &nbsp;&nbsp;desc: 'Demo group'
    }`"]

  classDef leftalign text-align:left
  class ev1 leftalign
```

Comme l'erreur a été notifiée, quelqu'un corrige le nom du groupe dans la source de données. Cette modification entraîne un événement *modified* correspondant. Cet événement *modified* ne sera pas traité, mais ajouté à la file d'erreurs puisque son objet a déjà un événement dans la file d'erreurs.

- sans auto-remédiation, tant que le premier événement n'aura pas été traité avec succès, le deuxième ne sera même pas tenté. La correction est bloquée.
- avec l'auto-remédiation, la file d'erreurs fusionne les deux événements, puis lors du prochain traitement de la file d'erreurs, l'événement résultant sera traité avec succès.

``` mermaid
flowchart TB
  subgraph errorqueuebis [avec auto remédiation]
    direction TB
    ev1bis
  end

  subgraph errorqueue [Sans auto remédiation]
    direction TB
    ev1
    ev2
  end

  ev1["`**événement 1**
    &nbsp;
    *eventtype*: added
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;grp_pkey: 42
    &nbsp;&nbsp;name: 'NomInvalide.'
    &nbsp;&nbsp;desc: 'Demo group'
    }`"]

  ev2["`**événement 2**
    &nbsp;
    *eventtype*: modified
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;modified: {
    &nbsp;&nbsp;&nbsp;&nbsp;name: 'NomValide'
    &nbsp;&nbsp;}
    }`"]

  ev1bis["`**événement 1**
    &nbsp;
    *eventtype*: added
    *objType*: ADGroup
    *objpkey*: 42
    *objattrs*: {
    &nbsp;&nbsp;grp_pkey: 42
    &nbsp;&nbsp;name: 'NomValide'
    &nbsp;&nbsp;desc: 'Demo group'
    }`"]

  classDef leftalign text-align:left
  class ev1,ev2,ev1bis leftalign
```
