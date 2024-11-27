---
title: Plugins
weight: 1
ordersectionsby: title
---

Quel que soit son type, un plugin est toujours un dossier nommé '*plugin_name*' contenant au moins les 4 fichiers suivants :

## Code source du plugin

Hermes va essayer d'importer le fichier ***plugin_name*.py**. Il est possible de découper le code du plugin en plusieurs fichiers et dossiers, mais le plugin sera toujours importé à partir de ce fichier.

Pour plus de détails sur l'API du plugin, veuillez consulter les sections suivantes :

- [Attributs](/development/plugins/attributes/)
- [Clients](/development/plugins/clients/)
- [Sources de données](/development/plugins/datasources/)
- [Consommateurs de bus de messages](/development/plugins/messagebus_consumers/)
- [Producteurs de bus de messages](/development/plugins/messagebus_producers/)

{{% notice tip %}}
Certains modules utilitaires sont disponibles dans `helpers` :

- `helpers.command` : pour exécuter des commandes locales sur l'hôte du client
- `helpers.ldaphashes`: pour générer les hachages LDAP à partir de mots de passe en clair
- `helpers.randompassword` : pour générer des mots de passe aléatoires avec des contraintes spécifiques
{{% /notice %}}

## Schéma de configuration du plugin

Selon le type de plugin, le fichier de schéma de configuration varie légèrement.

### Schéma de configuration du plugin pour les plugins clients

Hermes va essayer de valider la configuration du plugin avec un [schéma de validation Cerberus](https://docs.python-cerberus.org/schemas.html) spécifié dans un fichier YAML : **config-schema-client-*plugin_name*.yml**.

Le fichier de validation des plugins clients doit être vide ou ne contenir qu'une seule clé de premier niveau qui doit être le nom du plugin préfixé par `hermes-client-`.

Exemple pour le nom du plugin `usersgroups_flatfiles_emails_of_groups` :

```yaml { title="config-schema-client-usersgroups_flatfiles_emails_of_groups.yml" }
# https://docs.python-cerberus.org/validation-rules.html

hermes-client-usersgroups_flatfiles_emails_of_groups:
  type: dict
  required: true
  empty: false
  schema:
    destDir:
      type: string
      required: true
      empty: false
    onlyTheseGroups:
      type: list
      required: true
      nullable: false
      default: []
      schema:
        type: string
```

### Schéma de configuration du plugin pour les autres types de plugins

Hermes va essayer de valider la configuration du plugin avec un [schéma de validation Cerberus](https://docs.python-cerberus.org/schemas.html) spécifié dans un fichier YAML : **config-schema-plugin-*plugin_name*.yml**.

Même si le plugin ne nécessite aucune configuration, il nécessite tout de même un fichier de validation vide.

Exemple pour le nom de plugin `ldapPasswordHash` :

```yaml { title="config-schema-plugin-ldapPasswordHash.yml" }
# https://docs.python-cerberus.org/validation-rules.html

default_hash_types:
  type: list
  required: false
  nullable: false
  empty: true
  default: []
  schema:
    type: string
    allowed:
      - MD5
      - SHA
      - SMD5
      - SSHA
      - SSHA256
      - SSHA512

```

## Fichier README.md du plugin

La documentation doit être écrite dans **README.md** et doit contenir les sections suivantes :

```md { title="README.md" }
# `plugin_name` attribute plugin

## Description

## Configuration

## Usage
Uniquement pour les plugins `attributes` et `datasources`.

## Datamodel
Uniquement pour les plugins `clients`.
```

## Dépendances du plugin : requirements.txt

Même si le plugin n'a pas de dépendance Python, veuillez créer un fichier pip `requirements.txt` commençant par un commentaire contenant le chemin du plugin et se terminant par une ligne vide.

Exemple :

```txt { title="requirements.txt" }
# plugins/attributes/crypto_RSA_OAEP
pycryptodomex==3.21.0
 
```
