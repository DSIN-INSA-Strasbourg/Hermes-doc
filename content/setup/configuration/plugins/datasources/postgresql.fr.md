---
title: postgresql
---

## Description

Ce plugin permet d'utiliser une base de données PostgreSQL comme source de données.

## Configuration

Les paramètres de connexion sont requis dans la configuration du plugin.

```yaml
hermes:
  plugins:
    datasources:
      # Nom de la source. Utilisez ce que vous voulez. Sera utilisé dans le modèle de données
      your_source_name:
        type: postgresql
        settings:
          # OBLIGATOIRE : le nom DNS ou l'adresse IP du serveur de base de données
          server: dummy.example.com
          # OBLIGATOIRE : le port de connexion à la base de données
          port: 1234
          # MANDATORY: le nom de la base de données
          dbname: DUMMY
          # OBLIGATOIRE : les identifiants de connexion à la base de données
          login: HERMES_DUMMY
          password: "DuMmY_p4s5w0rD"
```

## Utilisation

Spécifiez une requête. Si vous souhaitez utiliser des valeurs provenant du cache, il est possible de les indiquer dans un dictionnaire `vars`, et y faire référence en spécifiant le nom de variable (clé) entouré par `%()s` dans la requête : cela nettoiera automatiquement les données dans la requête pour limiter les risques d'injection SQL. Voir l'exemple ci-dessous.

Les noms d'exemple dans `vars` sont préfixés par `sanitized_` pour donner plus de clarté, mais cela n'a rien d'obligatoire.

```yaml
hermes-server:
  datamodel:
    oneDataType:
      sources:
        your_source_name: # 'your_source_name' a été défini dans les paramètres du plugin
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM A_POSTGRESQL_TABLE

          commit_one:
            type: modify
            query: >-
              UPDATE A_POSTGRESQL_TABLE
              SET
                valueToSet = %(sanitized_valueToSet)s
              WHERE pkey = %(sanitized_pkey)s

            vars:
              sanitized_pkey: "{{ ITEM_FETCHED_VALUES.pkey }}"
              sanitized_valueToSet: "{{ ITEM_FETCHED_VALUES.valueToSet }}"
```