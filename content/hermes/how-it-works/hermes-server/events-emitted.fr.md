---
title: Événements émis
weight: 5
---

### Catégories d'événements

Un événement appartient toujours à l'une de ces catégories :

- `base`: événement standard, peut être de type :
  - `dataschema`: propage le nouveau schéma de données aux clients, après une mise à jour du modèle de données du serveur
  - `added`: une nouvelle entrée a été ajoutée au type de données spécifié, avec des attributs et des valeurs spécifiés
  - `removed`: l'entrée de la clé primaire spécifiée a été supprimée du type de données spécifié
  - `modified`: l'entrée de la clé primaire spécifiée a été modifiée. Contient uniquement les attributs ajoutés, modifiés et supprimés avec leurs nouvelles valeurs

- `initsync`: indique que l'événement fait partie d'une séquence initsync, peut être de type :
  - `init-start`: début d'une séquence initsync, contient également le schéma de données actuel
  - `added`: une nouvelle entrée a été ajoutée au type de données spécifié, avec les attributs et les valeurs spécifiés. Lorsque le serveur envoie le contenu de son cache pour initialiser les clients, les entrées ne peuvent qu'être ajoutées
  - `init-stop`: fin d'une séquence initsync
