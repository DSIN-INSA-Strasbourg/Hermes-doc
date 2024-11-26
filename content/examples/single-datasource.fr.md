---
title: 01. Source de données unique
weight: 1
---
## Contexte

Dans cet exemple, nous disposons d'une unique source de données (une base de données Oracle) que nous utiliserons pour gérer le cas classique des utilisateurs, mots de passe, groupes et l'appartenance des utilisateurs aux groupes, dans le but de remplir un serveur LDAP.

### Schéma Oracle

```mermaid
classDiagram
    direction BT
    ORA_USERPASSWORDS <-- ORA_USERS
    ORA_GROUPSMEMBERS <-- ORA_USERS
    ORA_GROUPSMEMBERS <-- ORA_GROUPS
    class ORA_USERS{
      USER_ID - NUMBER, NOT NULL
      LOGIN - VARCHAR2
      FIRSTNAME - VARCHAR2
      LASTNAME - VARCHAR2
      EMAIL - VARCHAR2
    }
    class ORA_USERPASSWORDS{
      USER_ID - NUMBER, NOT NULL
      PASSWORD_ENCRYPTED - RAW
      LDAP_HASHES - VARCHAR2
    }
    class ORA_GROUPS{
      GROUP_ID - NUMBER, NOT NULL
      GROUP_NAME - VARCHAR2
      GROUP_DESC - VARCHAR2
    }
    class ORA_GROUPSMEMBERS{
      USER_ID - NUMBER, NOT NULL
      GROUP_ID - NUMBER, NOT NULL
    }
```

### hermes-server-config

{{< tabs groupid="hermes-server-config">}}
{{% tab title="Complet avec commentaires" %}}

```yaml { wrap="false" }
hermes:
  cache:
    dirpath: /path/to/.hermes/hermes-server/cache
    enable_compression: true
    backup_count: 1
  cli_socket:
    path: /path/to/.hermes/hermes-server.sock # Facultatif, requis pour utiliser la CLI
    owner: user_login # Facultatif
    group: group_name # Facultatif
    # Facultatif, '0600' par défaut.
    # La valeur DOIT être préfixée par un 0 pour indiquer qu'il s'agit d'un entier octal
    mode: 0660
  logs:
    logfile: /path/to/.hermes/hermes-server/logs/hermes-server.log
    backup_count: 31 # 1 mois
    verbosity: info
  mail:
    server: dummy.example.com
    from: Hermes Server <no-reply@example.com>
    to:
      - user@example.com
  plugins:
    # Plugins d'attribut (filtres jinja)
    attributes:
      ldapPasswordHash:
        settings:
          default_hash_types:
            - SMD5
            - SSHA
            - SSHA256
            - SSHA512

      crypto_RSA_OAEP:
        settings:
          keys:
            decrypt_from_datasource:
              hash: SHA256
              # ATTENTION - CETTE CLÉ EST FAIBLE ET PUBLIQUE, NE L'UTILISEZ JAMAIS
              rsa_key: |-
                -----BEGIN RSA PRIVATE KEY-----
                MIGrAgEAAiEAstltWwDzmtSSHi7lfKqtUIO4dI8aX/EAopNdR/cWXH8CAwEAAQIh
                AKfflFjGNOJQwvJX3Io+/juxO+HFd7SRC++zBD9paZqZAhEA5OtjZQUapRrV/aC5
                NXFsswIRAMgBtgpz+t0FxyEXdzlcTwUCEHU6WZ8M2xU7xePpH49Ps2MCEQC+78s+
                /WvfNtXcRI+gJfyVAhAjcIWzHC5q4wzgL7psbPGy
                -----END RSA PRIVATE KEY-----

    # SERVEUR UNIQUEMENT - Sources utilisées pour récupérer les données. Au moins une doit être définie
    datasources:
      datasource_of_example1: # Nom de la source. Utilisez ce que vous voulez. Sera utilisé dans le modèle de données
        type: oracle # Type de source. Un plugin de source de données portant ce nom doit exister
        settings: # Paramètres de la source actuelle
          login: HERMES_DUMMY
          password: "DuMmY_p4s5w0rD"
          port: 1234
          server: dummy.example.com
          sid: DUMMY

    messagebus:
      kafka:
        settings:
          servers:
            - dummy.example.com:9093
          ssl:
            certfile: /path/to/.hermes/dummy.crt
            keyfile: /path/to/.hermes/dummy.pem
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt
          topic: hermes

hermes-server:
  updateInterval: 60 # Intervalle entre deux actualisations des données, en secondes

  # L'ordre de déclaration des types de données est important :
  # - les événements d'ajout/modification seront traités dans l'ordre de déclaration
  # - les événements de suppression seront traités dans l'ordre de déclaration inverse
  datamodel:
    SRVGroups: # Paramètres pour le type de données SRVGroups
      primarykeyattr: srv_group_id # Nom de l'attribut du modèle de données utilisé comme clé primaire
      # Facultatif : template Jinja permettant de définir la représentation d'une entrée dans les fichiers journaux
      toString: "<SRVGroups[{{ srv_group_id }}, {{ srv_group_name | default('#UNDEF#') }}]>"
      # Sources de données à utiliser pour récupérer les données.
      # Dans la plupart des cas il n'y en a qu'une seule, mais plusieurs peuvent être utilisées
      sources:
        datasource_of_example1: # Le nom de la source défini dans hermes.plugins.datasources
          # La requête permettant de récupérer les données.
          # 'type' est obligatoire et indique au plugin quel type de requête exécuter
          # Les valeurs 'type' possibles sont 'add', 'delete', 'fetch' et 'modify'
          # 'query' contient la requête à exécuter
          # 'vars' est un dictionnaire avec des variables à utiliser (et à nettoyer !) dans la requête
          #
          # Selon le plugin de source de données utilisé, 'query' et 'vars' peuvent être facultatifs.
          # Un template Jinja peut être inséré dans les valeurs 'query' et 'vars' pour éviter
          # d'utiliser des wildcards, de saisir manuellement la liste d'attributs, ou pour filtrer la
          # requête à l'aide d'une valeur mise en cache.
          #
          # Les variables Jinja disponibles sont [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # Voir la documentation pour plus de détails :
          # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_GROUPS
          attrsmapping:
            srv_group_id: GROUP_ID
            srv_group_name: GROUP_NAME
            srv_group_desc: GROUP_DESC

    SRVUsers: # Paramètres pour le type de données SRVUsers
      primarykeyattr: srv_user_id # Nom de l'attribut du modèle de données utilisé comme clé primaire
      # Facultatif : template Jinja permettant de définir la représentation d'une entrée dans les fichiers journaux
      toString: "<SRVUsers[{{ srv_user_id }}, {{ srv_login | default('#UNDEF#') }}]>"
      # Sources de données à utiliser pour récupérer les données.
      # Dans la plupart des cas il n'y en a qu'une seule, mais plusieurs peuvent être utilisées
      sources:
        datasource_of_example1: # Le nom de la source défini dans hermes.plugins.datasources
          # La requête permettant de récupérer les données.
          # 'type' est obligatoire et indique au plugin quel type de requête exécuter
          # Les valeurs 'type' possibles sont 'add', 'delete', 'fetch' et 'modify'
          # 'query' contient la requête à exécuter
          # 'vars' est un dictionnaire avec des variables à utiliser (et à nettoyer !) dans la requête
          #
          # Selon le plugin de source de données utilisé, 'query' et 'vars' peuvent être facultatifs.
          # Un template Jinja peut être inséré dans les valeurs 'query' et 'vars' pour éviter
          # d'utiliser des wildcards, de saisir manuellement la liste d'attributs, ou pour filtrer la
          # requête à l'aide d'une valeur mise en cache.
          #
          # Les variables Jinja disponibles sont [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # Voir la documentation pour plus de détails :
          # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_USERS

          attrsmapping:
            srv_user_id: USER_ID
            srv_login: LOGIN
            # Garantit que la première lettre de chaque nom sera en majuscule et que les autres seront en minuscules
            srv_firstname: "{{ FIRSTNAME | title}}"
            srv_lastname: "{{ LASTNAME | title}}"
            srv_mail: MAIL

    SRVUserPasswords: # Paramètres pour le type de données SRVUserPasswords
      primarykeyattr: srv_user_id # Nom de l'attribut du modèle de données utilisé comme clé primaire

      # Contraintes d'intégrité entre les types de données, en Jinja.
      # ATTENTION : cela peut être affreusement lent, vous devriez donc essayer de faire
      # au plus simple et idéalement vous limiter aux clés primaires.
      # Les variables Jinja disponibles sont '_SELF' : l'objet actuel, et chaque type de données déclaré
      # Pour chaque "typename" declaré, deux variables sont disponibles :
      # - typename_pkeys: un ensemble (set) contenant chaque clé primaire du type de données spécifié
      # - typename: une liste de dictionnaires contenant chaque entrée du type de données spécifié
      # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
      integrity_constraints:
        - "{{ _SELF.srv_user_id in SRVUsers_pkeys }}"
      
      # Sources de données à utiliser pour récupérer les données.
      # Dans la plupart des cas il n'y en a qu'une seule, mais plusieurs peuvent être utilisées
      sources:
        datasource_of_example1: # Le nom de la source défini dans hermes.plugins.datasources
          # La requête permettant de récupérer les données.
          # 'type' est obligatoire et indique au plugin quel type de requête exécuter
          # Les valeurs 'type' possibles sont 'add', 'delete', 'fetch' et 'modify'
          # 'query' contient la requête à exécuter
          # 'vars' est un dictionnaire avec des variables à utiliser (et à nettoyer !) dans la requête
          #
          # Selon le plugin de source de données utilisé, 'query' et 'vars' peuvent être facultatifs.
          # Un template Jinja peut être inséré dans les valeurs 'query' et 'vars' pour éviter
          # d'utiliser des wildcards, de saisir manuellement la liste d'attributs, ou pour filtrer la
          # requête à l'aide d'une valeur mise en cache.
          #
          # Les variables Jinja disponibles sont [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # Voir la documentation pour plus de détails :
          # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
          fetch:
            type: fetch
            query: >-
              SELECT p.{{ REMOTE_ATTRIBUTES | join(', p.') }}
              FROM ORA_USERPASSWORDS p

          # Pour chaque entrée traitée avec succès, nous supprimerons PASSWORD_ENCRYPTED
          # et stockerons les LDAP_HASHES fraîchement calculés.
          #
          # Facultatif. Une requête à exécuter à chaque fois qu'une entrée a été traitée.
          # sans erreur.
          # 'type' est obligatoire et indique au plugin quel type de requête exécuter
          # Les valeurs 'type' possibles sont 'add', 'delete', 'fetch' et 'modify'
          # 'query' contient la requête à exécuter
          # 'vars' est un dictionnaire avec des variables à utiliser (et à nettoyer !) dans la requête
          #
          # Selon le plugin de source de données utilisé, 'query' et 'vars' peuvent être facultatifs.
          # Un template Jinja peut être inséré dans les valeurs 'query' et 'vars' pour éviter
          # d'utiliser des wildcards, de saisir manuellement la liste d'attributs, ou pour filtrer la
          # requête à l'aide d'une valeur mise en cache.
          #
          # Les variables Jinja disponibles sont [REMOTE_ATTRIBUTES, ITEM_CACHED_VALUES, ITEM_FETCHED_VALUES].
          # Voir la documentation pour plus de détails :
          # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one
          commit_one:
            type: modify
            query: >-
              UPDATE ORA_USERPASSWORDS
              SET
                PASSWORD_ENCRYPTED = NULL,
                LDAP_HASHES = :ldap_hashes
              WHERE USER_ID = :user_id

            vars:
              user_id: "{{ ITEM_FETCHED_VALUES.srv_user_id }}"
              ldap_hashes: "{{ ';'.join(ITEM_FETCHED_VALUES.srv_password_ldap) }}"

          attrsmapping:
            srv_user_id: USER_ID
            # Déchiffre la valeur PASSWORD_ENCRYPTED pour générer les hachages LDAP.
            srv_password_ldap: >-
              {{
                (
                  PASSWORD_ENCRYPTED
                  | crypto_RSA_OAEP('decrypt_from_datasource')
                  | ldapPasswordHash
                )
                | default(None if LDAP_HASHES is None else LDAP_HASHES.split(';'))
              }}

    SRVGroupsMembers:
      # Nom de l'attribut du modèle de données utilisé comme clé primaire : un tuple dans le cas présent
      primarykeyattr: [srv_group_id, srv_user_id]
      # Déclaration des clés étrangères entre des types de données
      # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.foreignkeys
      foreignkeys:
        srv_group_id:
          from_objtype: SRVGroups
          from_attr: srv_group_id
        srv_user_id:
          from_objtype: SRVUsers
          from_attr: srv_user_id
      # Contraintes d'intégrité entre les types de données, en Jinja.
      # ATTENTION : cela peut être affreusement lent, vous devriez donc essayer de faire
      # au plus simple et idéalement vous limiter aux clés primaires.
      # Les variables Jinja disponibles sont '_SELF' : l'objet actuel, et chaque type de données déclaré
      # Pour chaque "typename" declaré, deux variables sont disponibles :
      # - typename_pkeys: un ensemble (set) contenant chaque clé primaire du type de données spécifié
      # - typename: une liste de dictionnaires contenant chaque entrée du type de données spécifié
      # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
      integrity_constraints:
        - "{{ _SELF.srv_user_id in SRVUsers_pkeys and _SELF.srv_group_id in SRVGroups_pkeys }}"
      # Sources de données à utiliser pour récupérer les données.
      # Dans la plupart des cas il n'y en a qu'une seule, mais plusieurs peuvent être utilisées
      sources:
        datasource_of_example1: # Le nom de la source défini dans hermes.plugins.datasources
          # La requête permettant de récupérer les données.
          # 'type' est obligatoire et indique au plugin quel type de requête exécuter
          # Les valeurs 'type' possibles sont 'add', 'delete', 'fetch' et 'modify'
          # 'query' contient la requête à exécuter
          # 'vars' est un dictionnaire avec des variables à utiliser (et à nettoyer !) dans la requête
          #
          # Selon le plugin de source de données utilisé, 'query' et 'vars' peuvent être facultatifs.
          # Un template Jinja peut être inséré dans les valeurs 'query' et 'vars' pour éviter
          # d'utiliser des wildcards, de saisir manuellement la liste d'attributs, ou pour filtrer la
          # requête à l'aide d'une valeur mise en cache.
          #
          # Les variables Jinja disponibles sont [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # Voir la documentation pour plus de détails :
          # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_GROUPSMEMBERS
          attrsmapping:
            srv_user_id: USER_ID
            srv_group_id: GROUP_ID
```

{{% /tab %}}
{{% tab title="Minimaliste" %}}

```yaml { wrap="false" }
hermes:
  cache:
    dirpath: /path/to/.hermes/hermes-server/cache
  cli_socket:
    path: /path/to/.hermes/hermes-server.sock
  logs:
    logfile: /path/to/.hermes/hermes-server/logs/hermes-server.log
    verbosity: info
  mail:
    server: dummy.example.com
    from: Hermes Server <no-reply@example.com>
    to:
      - user@example.com
  plugins:
    attributes:
      ldapPasswordHash:
        settings:
          default_hash_types:
            - SMD5
            - SSHA
            - SSHA256
            - SSHA512

      crypto_RSA_OAEP:
        settings:
          keys:
            decrypt_from_datasource:
              hash: SHA256
              # ATTENTION - CETTE CLÉ EST FAIBLE ET PUBLIQUE, NE L'UTILISEZ JAMAIS
              rsa_key: |-
                -----BEGIN RSA PRIVATE KEY-----
                MIGrAgEAAiEAstltWwDzmtSSHi7lfKqtUIO4dI8aX/EAopNdR/cWXH8CAwEAAQIh
                AKfflFjGNOJQwvJX3Io+/juxO+HFd7SRC++zBD9paZqZAhEA5OtjZQUapRrV/aC5
                NXFsswIRAMgBtgpz+t0FxyEXdzlcTwUCEHU6WZ8M2xU7xePpH49Ps2MCEQC+78s+
                /WvfNtXcRI+gJfyVAhAjcIWzHC5q4wzgL7psbPGy
                -----END RSA PRIVATE KEY-----

    datasources:
      datasource_of_example1:
        type: oracle
        settings:
          login: HERMES_DUMMY
          password: "DuMmY_p4s5w0rD"
          port: 1234
          server: dummy.example.com
          sid: DUMMY

    messagebus:
      kafka:
        settings:
          servers:
            - dummy.example.com:9093
          ssl:
            certfile: /path/to/.hermes/dummy.crt
            keyfile: /path/to/.hermes/dummy.pem
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt
          topic: hermes

hermes-server:
  # L'ordre de déclaration des types de données est important :
  # - les événements d'ajout/modification seront traités dans l'ordre de déclaration
  # - les événements de suppression seront traités dans l'ordre de déclaration inverse
  datamodel:
    SRVGroups:
      primarykeyattr: srv_group_id
      toString: "<SRVGroups[{{ srv_group_id }}, {{ srv_group_name | default('#UNDEF#') }}]>"
      sources:
        datasource_of_example1:
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_GROUPS
          attrsmapping:
            srv_group_id: GROUP_ID
            srv_group_name: GROUP_NAME
            srv_group_desc: GROUP_DESC

    SRVUsers:
      primarykeyattr: srv_user_id
      toString: "<SRVUsers[{{ srv_user_id }}, {{ srv_login | default('#UNDEF#') }}]>"
      sources:
        datasource_of_example1:
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_USERS

          attrsmapping:
            srv_user_id: USER_ID
            srv_login: LOGIN
            # Garantit que la première lettre de chaque nom sera en majuscule et que les autres seront en minuscules
            srv_firstname: "{{ FIRSTNAME | title}}"
            srv_lastname: "{{ LASTNAME | title}}"
            srv_mail: MAIL

    SRVUserPasswords:
      primarykeyattr: srv_user_id

      # Contraintes d'intégrité entre les types de données, en Jinja.
      # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
      integrity_constraints:
        - "{{ _SELF.srv_user_id in SRVUsers_pkeys }}"
      
      sources:
        datasource_of_example1:
          fetch:
            type: fetch
            query: >-
              SELECT p.{{ REMOTE_ATTRIBUTES | join(', p.') }}
              FROM ORA_USERPASSWORDS p

          # Pour chaque entrée traitée avec succès, nous supprimerons PASSWORD_ENCRYPTED
          # et stockerons les LDAP_HASHES fraîchement calculés.
          # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one
          commit_one:
            type: modify
            query: >-
              UPDATE ORA_USERPASSWORDS
              SET
                PASSWORD_ENCRYPTED = NULL,
                LDAP_HASHES = :ldap_hashes
              WHERE USER_ID = :user_id

            vars:
              user_id: "{{ ITEM_FETCHED_VALUES.srv_user_id }}"
              ldap_hashes: "{{ ';'.join(ITEM_FETCHED_VALUES.srv_password_ldap) }}"

          attrsmapping:
            srv_user_id: USER_ID
            # Déchiffre la valeur PASSWORD_ENCRYPTED pour générer les hachages LDAP.
            srv_password_ldap: >-
              {{
                (
                  PASSWORD_ENCRYPTED
                  | crypto_RSA_OAEP('decrypt_from_datasource')
                  | ldapPasswordHash
                )
                | default(None if LDAP_HASHES is None else LDAP_HASHES.split(';'))
              }}

    SRVGroupsMembers:
      # La clé primaire est un tuple
      primarykeyattr: [srv_group_id, srv_user_id]
      foreignkeys:
        srv_group_id:
          from_objtype: SRVGroups
          from_attr: srv_group_id
        srv_user_id:
          from_objtype: SRVUsers
          from_attr: srv_user_id
      # Contraintes d'intégrité entre les types de données, en Jinja.
      # https://hermes.insa-strasbourg.fr/fr/setup/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
      integrity_constraints:
        - "{{ _SELF.srv_user_id in SRVUsers_pkeys and _SELF.srv_group_id in SRVGroups_pkeys }}"
      sources:
        datasource_of_example1:
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_GROUPSMEMBERS
          attrsmapping:
            srv_user_id: USER_ID
            srv_group_id: GROUP_ID

```

{{% /tab %}}
{{< /tabs >}}

### hermes-client-usersgroups_ldap-config

```yaml { title="hermes-client-usersgroups_ldap-config.yml" wrap="false" }
hermes:
  cache:
    dirpath: /path/to/.hermes/hermes-client-usersgroups_ldap/cache
  cli_socket:
    path: /path/to/.hermes/hermes-client-usersgroups_ldap.sock
  logs:
    logfile: /path/to/.hermes/hermes-client-usersgroups_ldap/logs/hermes-client-usersgroups_ldap.log
    verbosity: info
  mail:
    server: dummy.example.com
    from: hermes-client-usersgroups_ldap <no-reply@example.com>
    to:
      - user@example.com
  plugins:
    messagebus:
      kafka:
        settings:
          servers:
            - dummy.example.com:9093
          ssl:
            certfile: /path/to/.hermes/dummy.crt
            keyfile: /path/to/.hermes/dummy.pem
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt
          topic: hermes
          group_id: hermes-grp

hermes-client-usersgroups_ldap:
    uri: ldaps://ldap.example.com:636
    binddn: cn=account,dc=example,dc=com
    bindpassword: s3cReT_p4s5w0rD
    basedn: dc=example,dc=com
    users_ou: ou=users,dc=example,dc=com
    groups_ou: ou=groups,dc=example,dc=com
    
    # OBLIGATOIRE : nom de l'attribut DN pour les utilisateurs, les mots de passe utilisateur et les groupes
    # Vous devez définir des valeurs pour les trois, même si vous n'utilisez pas certains d'entre eux
    dnAttributes:
      Users: uid
      UserPasswords: uid
      Groups: cn
    
    propagateUserDNChangeOnGroupMember: true
    groupsObjectclass: groupOfNames

    # Il est possible de définir une valeur par défaut pour certains attributs pour les Users, UserPasswords et Groups
    # La valeur par défaut sera appliquée lors du traitements des événements added et modified, si l'attribut local n'a pas de valeur
    defaultValues:
      # Hack pour permettre la création d'un groupe vide, néessaire à cause du "MUST member" dans le schéma
      Groups:
        member: ""

    # Les attributs locaux répertoriés ici ne seront pas stockés dans LDAP pour les types Users, UserPasswords and Groups
    attributesToIgnore:
      Users:
        - user_pkey
      UserPasswords:
        - user_pkey
      Groups:
        - group_pkey

hermes-client:
  # Politique d'auto remédiation à appliquer dans la file d'erreurs pour les événements
  # concernant un même objet
  # - "disabled" : pas d'auto remédiation, les événements sont empilés tels quels (par défaut)
  # - "conservative" :
  #   - fusionne un événement added avec un événement modified suivant
  #   - fusionne deux événements modified successifs
  # - "maximum" :
  #   - fusionne un événement added avec un événement modified suivant
  #   - fusionne deux événements modified successifs
  #   - supprime les deux événements lorsqu'un événement added est suivi d'un événement removed
  #   - fusionne un événement removed suivi d'un événement added dans un événement modified
  #   - supprime un événement modified lorsqu'il est suivi d'un événement removed
  autoremediation: conservative

  datamodel:
    Users:
      hermesType: SRVUsers
      # Facultatif : template Jinja permettant de définir la représentation d'une entrée dans les fichiers journaux
      toString: "<Users[{{ user_pkey }}, {{ uid | default('#UNDEF#') }}]>"
      attrsmapping:
        user_pkey: srv_user_id
        uid: srv_login
        givenname: srv_firstname
        sn: srv_lastname
        mail: srv_mail
        # Compose le displayname avec deux autres attributs
        displayname: "{{ srv_firstname ~ ' ' ~  srv_lastname }}"
        #
        # Valeurs statiques
        # Les définir ici plutôt que dans les valeurs par défaut (defaultValues)
        # permettra de propager d'éventuelles modifications sur chaque entrée
        #
        objectclass: "{{ ['person', 'inetOrgPerson', 'eduPerson'] }}"

    UserPasswords:
      hermesType: SRVUserPasswords
      attrsmapping:
        user_pkey: srv_user_id
        userPassword: srv_password_ldap

    Groups:
      hermesType: SRVGroups
      toString: "<Groups[{{ group_pkey }}, {{ cn | default('#UNDEF#') }}]>"
      attrsmapping:
        group_pkey: srv_group_id
        cn: srv_group_name
        description: srv_group_desc
        #
        # Valeurs statiques
        # Les définir ici plutôt que dans les valeurs par défaut (defaultValues)
        # permettra de propager d'éventuelles modifications sur chaque entrée
        #
        objectclass: "{{ ['groupOfNames'] }}"

    GroupsMembers:
      hermesType: SRVGroupsMembers
      attrsmapping:
        # Le nom des clés 'user_pkey' et 'group_pkey' est imposé
        user_pkey: srv_user_id
        group_pkey: srv_group_id
```

## Flux d'attributs

```mermaid
flowchart LR
  subgraph Oracle
    direction LR
    ORA_GROUPS
    ORA_USERS
    ORA_USERPASSWORDS
    ORA_GROUPSMEMBERS
  end

  subgraph ORA_GROUPS
    direction LR
    ORA_GROUPS_GROUP_ID["GROUP_ID"]
    ORA_GROUPS_GROUP_NAME["GROUP_NAME"]
    ORA_GROUPS_GROUP_DESC["GROUP_DESC"]
  end

  subgraph ORA_USERS
    direction LR
    ORA_USERS_USER_ID["USER_ID"]
    ORA_USERS_LOGIN["LOGIN"]
    ORA_USERS_FIRSTNAME["FIRSTNAME"]
    ORA_USERS_LASTNAME["LASTNAME"]
    ORA_USERS_EMAIL["EMAIL"]
  end

  subgraph ORA_USERPASSWORDS
    direction LR
    ORA_USERPASSWORDS_USER_ID["USER_ID"]
    ORA_USERPASSWORDS_PASSWORD_ENCRYPTED["PASSWORD_ENCRYPTED"]
    ORA_USERPASSWORDS_LDAP_HASHES["LDAP_HASHES"]
  end

  subgraph ORA_GROUPSMEMBERS
    direction LR
    ORA_GROUPSMEMBERS_USER_ID["USER_ID"]
    ORA_GROUPSMEMBERS_GROUP_ID["GROUP_ID"]
  end



  subgraph hermes-server
    direction LR
    SRVGroups
    SRVUsers
    SRVUserPasswords
    SRVGroupsMembers
  end

  subgraph SRVGroups
    direction LR
    SRVGroups_srv_group_id["srv_group_id"]
    SRVGroups_srv_group_name["srv_group_name"]
    SRVGroups_srv_group_desc["srv_group_desc"]
  end
  ORA_GROUPS_GROUP_ID --> SRVGroups_srv_group_id
  ORA_GROUPS_GROUP_NAME --> SRVGroups_srv_group_name
  ORA_GROUPS_GROUP_DESC --> SRVGroups_srv_group_desc

  subgraph SRVUsers
    direction LR
    SRVUsers_srv_user_id["srv_user_id"]
    SRVUsers_srv_login["srv_login"]
    SRVUsers_srv_firstname["srv_firstname"]
    SRVUsers_srv_lastname["srv_lastname"]
    SRVUsers_srv_mail["srv_mail"]
  end
  ORA_USERS_USER_ID --> SRVUsers_srv_user_id
  ORA_USERS_LOGIN --> SRVUsers_srv_login
  ORA_USERS_FIRSTNAME -->|filtre Jinja 'title'| SRVUsers_srv_firstname
  ORA_USERS_LASTNAME -->|filtre Jinja 'title'| SRVUsers_srv_lastname
  ORA_USERS_EMAIL --> SRVUsers_srv_mail

  subgraph SRVUserPasswords
    direction LR
    SRVUserPasswords_srv_user_id["srv_user_id"]
    SRVUserPasswords_srv_password_ldap["srv_password_ldap"]
  end
  ORA_USERPASSWORDS_USER_ID --> SRVUserPasswords_srv_user_id
  ORA_USERPASSWORDS_PASSWORD_ENCRYPTED -->|"filtre Jinja  'crypto_RSA_OAEP | ldapPasswordHash'"| SRVUserPasswords_srv_password_ldap
  ORA_USERPASSWORDS_LDAP_HASHES <-->|LDAP_HASHED est rempli par, ou fournit sa valeur| SRVUserPasswords_srv_password_ldap

  subgraph SRVGroupsMembers
    direction LR
    SRVGroupsMembers_srv_user_id["srv_user_id"]
    SRVGroupsMembers_srv_group_id["srv_group_id"]
  end
  ORA_GROUPSMEMBERS_USER_ID --> SRVGroupsMembers_srv_user_id
  ORA_GROUPSMEMBERS_GROUP_ID --> SRVGroupsMembers_srv_group_id



  subgraph hermes-client-usersgroups_ldap
    direction LR
    ClientGroups
    ClientUsers
    ClientUserPasswords
    ClientGroupsMembers
  end

  subgraph ClientGroups
    direction LR
    ClientGroups_group_pkey["group_pkey"]
    ClientGroups_cn["cn"]
    ClientGroups_description["description"]
    ClientGroups_objectclass["objectclass"]
  end
  SRVGroups_srv_group_id --> ClientGroups_group_pkey
  SRVGroups_srv_group_name --> ClientGroups_cn
  SRVGroups_srv_group_desc --> ClientGroups_description
  
  subgraph ClientUsers
    direction LR
    ClientUsers_user_pkey["user_pkey"]
    ClientUsers_uid["uid"]
    ClientUsers_givenname["givenname"]
    ClientUsers_sn["sn"]
    ClientUsers_mail["mail"]
    ClientUsers_displayname["displayname"]
    ClientUsers_objectclass["objectclass"]
  end
  SRVUsers_srv_user_id --> ClientUsers_user_pkey
  SRVUsers_srv_login --> ClientUsers_uid
  SRVUsers_srv_firstname --> ClientUsers_givenname
  SRVUsers_srv_firstname --> ClientUsers_displayname
  SRVUsers_srv_lastname --> ClientUsers_displayname
  SRVUsers_srv_lastname --> ClientUsers_sn
  SRVUsers_srv_mail --> ClientUsers_mail
  
  subgraph ClientUserPasswords
    direction LR
    ClientUserPasswords_user_pkey["user_pkey"]
    ClientUserPasswords_userPassword["userPassword"]
  end
  SRVUserPasswords_srv_user_id --> ClientUserPasswords_user_pkey
  SRVUserPasswords_srv_password_ldap --> ClientUserPasswords_userPassword


  subgraph ClientGroupsMembers
    direction LR
    ClientGroupsMembers_user_pkey["user_pkey"]
    ClientGroupsMembers_group_pkey["group_pkey"]
  end
  SRVGroupsMembers_srv_user_id --> ClientGroupsMembers_user_pkey
  SRVGroupsMembers_srv_group_id --> ClientGroupsMembers_group_pkey




  subgraph LDAP
    direction LR
    LDAPGroups
    LDAPUsers
  end

  subgraph LDAPGroups
    direction LR
    LDAPGroups_cn["cn"]
    LDAPGroups_description["description"]
    LDAPGroups_objectclass["objectclass"]
    LDAPGroups_member["member"]
  end
  ClientGroups_cn --> LDAPGroups_cn
  ClientGroups_description --> LDAPGroups_description
  ClientGroups_objectclass --> LDAPGroups_objectclass
  ClientGroupsMembers_user_pkey -->|converti en DN d'utilisateur| LDAPGroups_member
  ClientGroupsMembers_group_pkey -->|converti en DN de groupe| LDAPGroups_member

  subgraph LDAPUsers
    direction LR
    LDAPUsers_uid["uid"]
    LDAPUsers_givenname["givenname"]
    LDAPUsers_displayname["displayname"]
    LDAPUsers_displayname["displayname"]
    LDAPUsers_sn["sn"]
    LDAPUsers_mail["mail"]
    LDAPUsers_objectclass["objectclass"]
    LDAPUsers_userPassword["userPassword"]
  end
  ClientUsers_uid --> LDAPUsers_uid
  ClientUsers_givenname --> LDAPUsers_givenname
  ClientUsers_displayname --> LDAPUsers_displayname
  ClientUsers_sn --> LDAPUsers_sn
  ClientUsers_mail --> LDAPUsers_mail
  ClientUsers_objectclass --> LDAPUsers_objectclass
  ClientUserPasswords_userPassword --> LDAPUsers_userPassword

  classDef global fill:#fafafa,stroke-dasharray: 5 5
  class Oracle,hermes-server,hermes-client-usersgroups_ldap,LDAP global
```
