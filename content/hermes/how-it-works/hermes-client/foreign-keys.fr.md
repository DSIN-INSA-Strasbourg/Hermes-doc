---
title: Clés étrangères
weight: 3
---

Il arrive que les objets soient liés entre eux par des clés étrangères. Lorsqu'une erreur survient sur un objet dont la clé primaire fait référence à celle d'un ou plusieurs autres objets "parents", il peut être souhaitable d'interrompre le traitement de tout ou partie des événements de ces objets parents jusqu'à ce que ce premier événement ait été correctement traité. Cela peut être réalisé en ajoutant les événements des objets parents à la file d'attente des erreurs au lieu de tenter de les traiter.

La première chose à faire est de déclarer les clés étrangères via [hermes-server.datamodel.data-type-name.foreignkeys](/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.foreignkeys) dans la configuration d'hermes-server. Le serveur ne fera rien d'autre avec ces clés étrangères que de les propager aux clients.

Ensuite, il faut définir la politique à appliquer aux clients via [hermes-client.foreignkeys_policy](/setup/configuration/hermes-client/#hermes-client.foreignkeys_policy) dans chaque configuration hermes-client. Il y en a trois possibles :

- `disabled` : Aucun événement, la politique est désactivée. Probablement pas pertinent dans la plupart des cas, mais pourrait peut-être être utile à quelqu'un un jour.
- `on_remove_event` : Uniquement sur les événements *removed*. Devrait suffire dans la plupart des cas.
- `on_every_event` : Sur tous les types d'événements (*added*, *modified*, *removed*). Pour assurer une cohérence parfaite quoi qu'il arrive.
