---
title: Clients
---

## Description

Un plugin client est simplement une classe fille de `GenericClient` conçue pour implémenter des gestionnaires d'événements simples et pour diviser leurs tâches en sous-tâches atomiques afin de garantir un retraitement cohérent en cas d'erreur.

## Conditions requises

Voici une implémentation de plugin minimale commentée qui ne fera rien, car elle n'implémente pas encore de gestionnaires d'événements.

```py
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

# Nécessaire pour hériter de la classe GenericClient
from clients import GenericClient

# Nécessaire pour les annotations de type des gestionnaires d'événements
from lib.config import HermesConfig # uniquement si le plugin implémente une méthode __init__()
from lib.datamodel.dataobject import DataObject
from typing import Any

# Nécessaire pour indiquer à hermes quelle classe il devra instancier
HERMES_PLUGIN_CLASSNAME = "MyPluginClassName"

class MyPluginClassName(GenericClient):
    def __init__(self, config: HermesConfig):
        # La variable 'config' ne doit être ni utilisée ni modifiée par le plugin
        super().__init__(config)
        # ... code d'initialisation du plugin
```

## Méthodes de gestion d'événements

### Gestionnaires d'événements

Pour chaque type de données configuré dans le modèle de données client, le plugin peut implémenter un gestionnaire pour chacun des 5 types d'événements possibles :

- `added` : lorsqu'un objet est ajouté
- `recycled` : lorsqu'un objet est restauré depuis la corbeille (ne sera jamais appelé si la corbeille est désactivée)
- `modified` : lorsqu'un objet est modifié
- `trashed` : lorsqu'un objet est placé dans la corbeille (ne sera jamais appelé si la corbeille est désactivée)
- `removed` : lorsqu'un objet est supprimé

Si un événement est reçu par un client, mais que son gestionnaire n'est pas implémenté, il sera ignoré silencieusement.

Chaque gestionnaire doit être nommé **on_*datatypename*_*eventtypename***.

Exemple pour un type de données `Mydatatype` :

```py
    def on_Mydatatype_added(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        pass

    def on_Mydatatype_recycled(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        pass

    def on_Mydatatype_modified(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
        cachedobj: DataObject,
    ):
        pass

    def on_Mydatatype_trashed(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        cachedobj: DataObject,
    ):
        pass

    def on_Mydatatype_removed(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        cachedobj: DataObject,
    ):
        pass
```

#### Arguments des gestionnaires d'événements

- `objkey` : la clé primaire de l'objet affecté par l'événement

- `eventattrs` : un dictionnaire contenant les nouveaux attributs de l'objet. Son contenu dépend du type d'événement :
  - `added` / `recycled` : contient tous les noms d'attributs comme clé, et leurs valeurs respectives comme valeur
    - `modified` : contient toujours trois clés :
      - `added` : attributs qui n'étaient pas définis auparavant, mais qui ont maintenant une valeur. Noms d'attributs comme clé, et leurs valeurs respectives comme valeur
      - `modified` : attributs qui étaient définis auparavant, mais dont la valeur a changé. Noms d'attributs comme clé, et leurs nouvelles valeurs respectives comme valeur
      - `removed` : attributs qui étaient définis auparavant, mais qui n'ont plus de valeur. Noms d'attributs comme clé, et `None` comme valeur
  - `trashed` / `removed` : toujours un dictionnaire vide `{}`

- `newobj` : une instance `DataObject` contenant toutes les valeurs mises à jour de l'objet affecté par l'événement (*voir [Instances de DataObject](#dataobject-instances) ci-dessous*)

- `cachedobj` : une instance `DataObject` contenant toutes les valeurs précédentes (mises en cache) de l'objet affecté par l'événement (*voir [Instances de DataObject](#dataobject-instances) ci-dessous*)

#### Instances de DataObject {#dataobject-instances}

Chaque objet de type de données peut être utilisé intuitivement via une instance `DataObject`.
Utilisons un exemple simple avec ces valeurs d'objet `User` (sans adresse email) du modèle de données ci-dessous :

```json
{
    "user_pkey": 42,
    "uid": "jdoe",
    "givenname": "John",
    "sn": "Doe"
}
```

```yaml
hermes-client:
  datamodel:
    Users:
      hermesType: SRVUsers
      attrsmapping:
        user_pkey: srv_user_id
        uid: srv_login
        givenname: srv_firstname
        sn: srv_lastname
        mail: srv_mail
```

Maintenant, si cet objet est stocké dans une instance de DataObject `newobj` :

```py
>>> newobj.getPKey()
42

>>> newobj.user_pkey
42

>>> newobj.uid
'jdoe'

>>> newobj.givenname
'John'

>>> newobj.sn
'Doe'

>>> newobj.mail
AttributeError: 'Users' object has no attribute 'mail'

>>> hasattr(newobj, 'sn')
True

>>> hasattr(newobj, 'mail')
False
```

#### Gestion d'erreur

Toute exception non gérée levée dans un gestionnaire d'événements sera gérée par `GenericClient`, qui ajoutera l'événement à sa file d'erreurs.
`GenericClient` essaiera alors de re-traiter l'événement régulièrement jusqu'à ce qu'il réussisse, et appellera donc le gestionnaire d'événements.

Mais parfois, un gestionnaire d'événements doit procéder à plusieurs opérations sur la cible. Imaginons un gestionnaire comme celui-ci :

```py
    def on_Mydatatype_added(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        if condition:
            operation1()  # "condition" vaut False, operation1() n'est pas appelée
        operation2()  # aucune erreur ne se produit
        operation3()  # celle-ci lève une exception
```

A chaque nouvelle tentative la fonction `operation2()` sera à nouveau appelée, mais ce n'est probablement pas souhaitable.

Il est possible de découper les étapes d'un gestionnaire d'événements en utilisant l'attribut `currentStep` hérité de `GenericClient`, afin de retenter les traitements des événements en erreur depuis l'étape qui avait échouée.

`currentStep` démarre toujours à 0 lors du traitement normal des événements. L'évolution de sa valeur doit être gérée par les plugins.

Lorsqu'une erreur se produit, la valeur `currentStep` est enregistrée dans la file d'erreurs avec l'événement.

Les re-tentatives de traitement de la file d'erreurs restaureront toujours la valeur de `currentStep` avant d'appeler le gestionnaire d'événements.

Ainsi en l'implémentant comme ci-dessous, `operation2()` ne sera appelée qu'une seule fois.

```py
    def on_Mydatatype_added(
        self,
        objkey: Any,
        eventattrs: "dict[str, Any]",
        newobj: DataObject,
    ):
        if self.currentStep == 0:
            if condition:
                operation1()  # "condition" vaut False, operation1() n'est pas appelée
                # Indique que des modifications ont été propagées sur la cible
                self.isPartiallyProcessed = True
            self.currentStep += 1
        
        if self.currentStep == 1:
            operation2()  # aucune erreur ne se produit
            # Indique que des modifications ont été propagées sur la cible
            self.isPartiallyProcessed = True
            self.currentStep += 1

        if self.currentStep == 2:
            operation3()  # celle-ci lève une exception
            # Indique que des modifications ont été propagées sur la cible
            self.isPartiallyProcessed = True
            self.currentStep += 1
```

##### Comprendre l'attribut `isPartiallyProcessed`

L'attribut `isPartiallyProcessed` hérité de `GenericClient` indique si le traitement de l'événement en cours a déjà propagé des modifications sur la cible. Il doit donc être mis à `True` dès que la moindre modification a été propagée sur la cible.
Il permet à l'autoremédiation de fusionner les événements dont `currentStep` est différent de 0 mais dont les étapes précédentes n'ont rien modifié sur la cible.

`isPartiallyProcessed` vaut toujours `False` lors d'un traitement d'événement normal. L'évolution de sa valeur doit être gérée par les plugins.

Avec l'exemple d'implémentation ci-dessus, et une exception levée par `operation3()`, l'autoremédiation ne tenterait pas de fusionner cet événement partiellement traité avec d'éventuels événements ultérieurs, car `isPartiallyProcessed` vaut `True`.

Avec l'exemple d'implémentation ci-dessus, mais une exception levée par `operation2()`, l'autoremédiation essaierait de fusionner cet événement non traité avec d'éventuels événements ultérieurs, car `isPartiallyProcessed` vaudrait encore `False`.

### Gestionnaire d'événement *on_save*

Un gestionnaire d'événements spécial qui peut être implémenté lorsque Hermes vient de sauvegarder ses fichiers cache : une fois que certains événements ont été traités et qu'aucun événement n'est en attente sur le bus de messages, ou avant la fin.

{{% notice warning %}}
Comme ce gestionnaire n'est pas un gestionnaire d'événements standard, `GenericClient` ne peut pas gérer les exceptions pour lui et procéder à une nouvelle tentative ultérieurement.

**Toute exception non gérée levée dans ce gestionnaire d'événements mettra immédiatement fin au client.**

Il appartient à l'implémentation d'éviter les erreurs.
{{% /notice %}}

```py
    def on_save(self):
        pass
```

## Propriétés et méthodes de GenericClient

## Propriétés

- ```py
  currentStep: int
  ```

  Numéro d'étape de l'événement en cours de traitement. Nécessaire pour permettre aux clients de reprendre un événement où il a échoué.

- ```py
  isPartiallyProcessed: bool
  ```

  Indique si le traitement de l'événement en cours a déjà propagé des modifications sur la cible.  
  Doit être défini à `True` dès que la moindre modification a été propagée sur la cible.  
  Il permet à l'autoremédiation de fusionner les événements dont `currentStep` est différent de 0 mais dont les étapes précédentes n'ont rien modifié sur la cible.

- ```py
  isAnErrorRetry: bool
  ```

  Attribut en lecture seule qui peut permettre au gestionnaire d'événements de savoir si l'événement actuel est en cours de traitement dans le cadre d'une nouvelle tentative suite à une erreur. Cela peut être utile par exemple pour effectuer des vérifications complémentaires lorsqu'une bibliothèque génère des exceptions même si elle a correctement traité les modifications demandées, comme le fait parfois python-ldap.

- ```py
  config: dict[str, Any]
  ```

  Dictionnaire contenant la configuration du plugin client.

## Méthodes

- ```py
  def getDataobjectlistFromCache(objtype: str) -> DataObjectList
  ```

  Renvoie le cache du type d'objet spécifié, par référence. Lève `IndexError` si `objtype` n'est pas valide
  {{% notice warning %}}
  Toute modification du contenu du cache corrompra votre client !!!
  {{% /notice %}}

- ```py
  def getObjectFromCache(objtype: str, objpkey: Any ) -> DataObject
  ```

  Renvoie une copie complète (deepcopy) d'un objet depuis le cache. Lève une erreur `IndexError` si `objtype` n'est pas valide ou si `objpkey` n'est pas trouvée

- ```py
  def mainLoop() -> None
  ```

  Boucle principale du client
  {{% notice warning %}}
  Appelée par Hermes, pour démarrer le client. **Ne doit jamais être appelée ni surchargée**
  {{% /notice %}}
