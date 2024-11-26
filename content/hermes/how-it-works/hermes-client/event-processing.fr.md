---
title: Traitement des événements
weight: 2
---

Comme le modèle de données sur le serveur diffère de celui sur le client, les clients doivent convertir les événements *distants* reçus sur le bus de messages en événements *locaux*. Si l'événement local résultant est vide (le type de données ou les attributs modifiés dans l'événement distant ne sont pas définis sur le modèle de données client), l'événement est ignoré.

Lors de la mise à jour du modèle de données client, le client peut générer des événements locaux qui n'ont pas d'événement distant correspondant, *i.e.* pour mettre à jour une valeur d'attribut calculée avec un modèle Jinja qui vient d'être modifié.

``` mermaid
flowchart TB
  subgraph Hermes-client
    direction TB
    datamodelUpdate[["mise à jour du modèle de données"]]
    remoteevent["Événement distant"]
    localevent["Événement local"]
    eventHandler(["Méthode de gestion de l'événement du plugin client"])
  end
  datamodelUpdate-->|génère|localevent
  MessageBus["Bus de message"]-->|fournit|remoteevent
  remoteevent-->|convertit en|localevent
  localevent-->|passe à la bonne|eventHandler
  eventHandler-->|traite|Target["Cible"]

  classDef external fill:#fafafa,stroke-dasharray: 5 5
  class MessageBus,Target external
```
