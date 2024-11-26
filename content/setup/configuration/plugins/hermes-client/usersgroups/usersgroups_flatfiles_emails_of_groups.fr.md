---
title: flatfiles_emails_of_groups
---

## Description

Ce client génére un fichier txt plat par `Groups`, contenant les adresses e-mail de ses membres (une par ligne).

## Configuration

```yaml
hermes-client-usersgroups_flatfiles_emails_of_groups:
  # OBLIGATOIRE
  destDir: "/path/where/files/are/stored"

  # Facultatif : si défini, générera un fichier uniquement pour les noms de groupe spécifiés dans cette liste
  onlyTheseGroups:
    - group1
    - group2
```

## Datamodel

Les types de données suivants doivent être configurés :

- `Users`, nécessite les noms d'attribut suivants :
  - `user_pkey` : la clé primaire de l'utilisateur
  - `mail` : l'adresse e-mail de l'utilisateur
- `Groups`, nécessite les noms d'attribut suivants :
  - `group_pkey` : la clé primaire du groupe
  - `name` : le nom du groupe, qui sera comparé à ceux de `onlyTheseGroups`, et utilisé pour nommer le fichier de destination "*groupName*.txt"
- `GroupsMembers`, nécessite les noms d'attribut suivants :
  - `user_pkey` : la clé primaire de l'utilisateur
  - `group_pkey` : la clé primaire du groupe

```yaml
  datamodel:
    Users:
      hermesType: your_server_Users_type_name
      attrsmapping:
        user_pkey: user_pkey_on_server
        mail: mail_on_server

    Groups:
      hermesType: your_server_Groups_type_name
      attrsmapping:
        group_pkey: group_pkey_on_server
        name: group_name_on_server

    GroupsMembers:
      hermesType: your_server_GroupsMembers_type_name
      attrsmapping:
        user_pkey: user_pkey_on_server
        group_pkey: group_pkey_on_server
```
