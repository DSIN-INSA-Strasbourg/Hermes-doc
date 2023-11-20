---
title: 01. Single datasource
url: single-datasource
weight: 41
---

## Context

In this example, we have an unique Datasource (an Oracle database) that we'll use to convert typical users, password, groups and group membership data to fill a LDAP server.

### Oracle schema

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
{{% tab title="Complete with comments" %}}

```yaml { wrap="false" }
hermes:
  cache:
    dirpath: /path/to/.hermes/hermes-server/cache
    enable_compression: true
    backup_count: 1
  cli_socket:
    path: /path/to/.hermes/hermes-server.sock # Facultative, required to use cli
    owner: user_login # Facultative
    group: group_name # Facultative
    # Facultative, '0600' will be used by default.
    # The value MUST be prefixed by a 0 to indicate that it's an octal integer
    mode: 0660
  logs:
    logfile: /path/to/.hermes/hermes-server/logs/hermes-server.log
    backup_count: 31 # 1 month
    verbosity: info
  mail:
    server: dummy.example.com
    from: Hermes Server <no-reply@example.com>
    to:
      - user@example.com
  plugins:
    # Attribute transform plugins (jinja filters)
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
              # WARNING - THIS KEY IS WEAK AND PUBLIC, NEVER USE IT
              rsa_key: |-
                -----BEGIN RSA PRIVATE KEY-----
                MIGrAgEAAiEAstltWwDzmtSSHi7lfKqtUIO4dI8aX/EAopNdR/cWXH8CAwEAAQIh
                AKfflFjGNOJQwvJX3Io+/juxO+HFd7SRC++zBD9paZqZAhEA5OtjZQUapRrV/aC5
                NXFsswIRAMgBtgpz+t0FxyEXdzlcTwUCEHU6WZ8M2xU7xePpH49Ps2MCEQC+78s+
                /WvfNtXcRI+gJfyVAhAjcIWzHC5q4wzgL7psbPGy
                -----END RSA PRIVATE KEY-----

    # SERVER ONLY - Sources used to fetch data. At lease one must be defined
    datasources:
      datasource_of_example1: # Source name. Use whatever you want. Will be used in datamodel
        type: oracle # Source type. A datasource plugin with this name must exist
        settings: # Settings of current source
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
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt
            certfile: /path/to/.hermes/dummy.crt
            keyfile: /path/to/.hermes/dummy.pem
          topic: hermes

hermes-server:
  updateInterval: 60 # Interval between two data update, in seconds

  # The declaration order of data types is important :
  # - add/modify events will be processed in the declaration order
  # - remove events will be processed in the reversed declaration order
  datamodel:
    SRVGroups: # Settings for SRVGroups's data type
      primarykeyattr: srv_group_id # Attribute's name that will be used as primary key
      # Template of object's string representation that will be used in logs
      toString: "<SRVGroups[{{ srv_group_id }}, {{ srv_group_name | default('#UNDEF#') }}]>"
      sources: # datasource(s) to use to fetch data. Usually one, but several could be used
        datasource_of_example1: # The source name set in hermes.plugins.datasources
          # The query to fetch data.
          # 'type' is mandatory and indicate to plugin which flavour of query to proceed
          #   Possible 'type' values are 'add', 'delete', 'fetch' and 'modify'
          # 'query' is the query to send
          # 'vars' is a dict with vars to use (and sanitize !) in query
          #
          # According to source type, 'query' and 'vars' may be facultative.
          # A Jinja template can be inserted in 'query' and 'vars' values to avoid wilcards
          # and manually typing the attribute's list, or to filter the query using a cached value.
          #
          # Jinja vars available are [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # See documentation for details :
          # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_GROUPS
          attrsmapping:
            srv_group_id: GROUP_ID
            srv_group_name: GROUP_NAME
            srv_group_desc: GROUP_DESC

    SRVUsers: # Settings for SRVUsers's data type
      primarykeyattr: srv_user_id # Attribute's name that will be used as primary key
      # Facultative template of object's string representation that will be used in logs
      toString: "<SRVUsers[{{ srv_user_id }}, {{ srv_login | default('#UNDEF#') }}]>"
      sources: # datasource(s) to use to fetch data. Usually one, but several could be used
        datasource_of_example1: # The source name set in hermes.plugins.datasources
          # The query to fetch data.
          # 'type' is mandatory and indicate to plugin which flavour of query to proceed
          #   Possible 'type' values are 'add', 'delete', 'fetch' and 'modify'
          # 'query' is the query to send
          # 'vars' is a dict with vars to use (and sanitize !) in query
          #
          # According to source type, 'query' and 'vars' may be facultative.
          # A Jinja template can be inserted in 'query' and 'vars' values to avoid wilcards
          # and manually typing the attribute's list, or to filter the query using a cached value.
          #
          # Jinja vars available are [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # See documentation for details :
          # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
          fetch:
            type: fetch
            query: >-
              SELECT {{ REMOTE_ATTRIBUTES | join(', ') }}
              FROM ORA_USERS

          attrsmapping:
            srv_user_id: USER_ID
            srv_login: LOGIN
             # Ensure first letter of each names is uppercase, and other are lowercase
            srv_firstname: "{{ FIRSTNAME | title}}"
            srv_lastname: "{{ LASTNAME | title}}"
            srv_mail: MAIL

    SRVUserPasswords: # Settings for SRVUserPasswords's data type
      primarykeyattr: srv_user_id # Attribute's name that will be used as primary key

      # Integrity constraints between datamodel's type, in Jinja.
      # WARNING : could be very slow, keep it as simple as possible, and focused upon
      # primary keys
      # Jinja vars available are '_SELF' : the current object, and every types declared
      # For each "typename" declared, two vars are available :
      # - typename_pkeys : a set with every primary keys
      # - typename : a list of dict containing each entries
      # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
      integrity_constraints:
        - "{{ _SELF.srv_user_id in SRVUsers_pkeys }}"
      
      sources: # datasource(s) to use to fetch data. Usually one, but several could be used
        datasource_of_example1: # The source name set in hermes.plugins.datasources
          # The query to fetch data.
          # 'type' is mandatory and indicate to plugin which flavour of query to proceed
          #   Possible 'type' values are 'add', 'delete', 'fetch' and 'modify'
          # 'query' is the query to send
          # 'vars' is a dict with vars to use (and sanitize !) in query
          #
          # According to source type, 'query' and 'vars' may be facultative.
          # A Jinja template can be inserted in 'query' and 'vars' values to avoid wilcards
          # and manually typing the attribute's list, or to filter the query using a cached value.
          #
          # Jinja vars available are [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # See documentation for details :
          # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
          fetch:
            type: fetch
            query: >-
              SELECT p.{{ REMOTE_ATTRIBUTES | join(', p.') }}
              FROM ORA_USERPASSWORDS p

          # For each entry successfully processed, we'll remove PASSWORD_ENCRYPTED
          # and store the freshly computed LDAP_HASHES.
          #
          # Facultative. The query to run each time an item of current data have been processed
          # without errors.
          # 'type' is mandatory and indicate to plugin which flavour of query to proceed
          #   Possible 'type' values are 'add', 'delete', 'fetch' and 'modify'
          # 'query' is the query to send
          # 'vars' is a dict with vars to use (and sanitize !) in query
          #
          # According to source type, 'query' and 'vars' may be facultative.
          # A Jinja template can be inserted in 'query' and 'vars' values to avoid wilcards
          # and manually typing the attribute's list, or to filter the query using a cached value.
          #
          # Jinja vars available are [REMOTE_ATTRIBUTES, ITEM_CACHED_VALUES, ITEM_FETCHED_VALUES].
          # See documentation for details :
          # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one
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
            # Decipher PASSWORD_ENCRYPTED value to generate the LDAP hashes.
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
      # Attribute's names that will be used as primary key : here is is a tuple
      primarykeyattr: [srv_group_id, srv_user_id]
      # Integrity constraints between datamodel's type, in Jinja.
      # WARNING : could be very slow, keep it as simple as possible, and focused upon
      # primary keys
      # Jinja vars available are '_SELF' : the current object, and every types declared
      # For each "typename" declared, two vars are available :
      # - typename_pkeys : a set with every primary keys
      # - typename : a list of dict containing each entries
      # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
      integrity_constraints:
        - "{{ _SELF.srv_user_id in SRVUsers_pkeys and _SELF.srv_group_id in SRVGroups_pkeys }}"
      sources: # datasource(s) to use to fetch data. Usually one, but several could be used
        datasource_of_example1: # The source name set in hermes.plugins.datasources
          # The query to fetch data.
          # 'type' is mandatory and indicate to plugin which flavour of query to proceed
          #   Possible 'type' values are 'add', 'delete', 'fetch' and 'modify'
          # 'query' is the query to send
          # 'vars' is a dict with vars to use (and sanitize !) in query
          #
          # According to source type, 'query' and 'vars' may be facultative.
          # A Jinja template can be inserted in 'query' and 'vars' values to avoid wilcards
          # and manually typing the attribute's list, or to filter the query using a cached value.
          #
          # Jinja vars available are [REMOTE_ATTRIBUTES, CACHED_VALUES].
          # See documentation for details :
          # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.fetch
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
{{% tab title="Minimalist" %}}

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
              # WARNING - THIS KEY IS WEAK AND PUBLIC, NEVER USE IT
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
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt
            certfile: /path/to/.hermes/dummy.crt
            keyfile: /path/to/.hermes/dummy.pem
          topic: hermes

hermes-server:
  # The declaration order of data types is important :
  # - add/modify events will be processed in the declaration order
  # - remove events will be processed in the reversed declaration order
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
             # Ensure first letter of each names is uppercase, and other are lowercase
            srv_firstname: "{{ FIRSTNAME | title}}"
            srv_lastname: "{{ LASTNAME | title}}"
            srv_mail: MAIL

    SRVUserPasswords:
      primarykeyattr: srv_user_id

      # Integrity constraints between datamodel's type, in Jinja.
      # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
      integrity_constraints:
        - "{{ _SELF.srv_user_id in SRVUsers_pkeys }}"
      
      sources:
        datasource_of_example1:
          fetch:
            type: fetch
            query: >-
              SELECT p.{{ REMOTE_ATTRIBUTES | join(', p.') }}
              FROM ORA_USERPASSWORDS p

          # For each entry successfully processed, we'll remove PASSWORD_ENCRYPTED
          # and store the freshly computed LDAP_HASHES.
          # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.sources.datasource-name.commit_one
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
            # Decipher PASSWORD_ENCRYPTED value to generate the LDAP hashes.
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
      # The primary key is a tuple
      primarykeyattr: [srv_group_id, srv_user_id]
      # Integrity constraints between datamodel's type, in Jinja.
      # https://hermes.insa-strasbourg.fr/en/configuration/hermes-server/#hermes-server.datamodel.data-type-name.integrity_constraints
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
          group_id: hermes-grp
          servers:
            - dummy.example.com:9093
          ssl:
            cafile: /path/to/.hermes/INTERNAL-CA-chain.crt
            certfile: /path/to/.hermes/dummy.crt
            keyfile: /path/to/.hermes/dummy.pem
          topic: hermes

hermes-client-usersgroups_ldap:
    uri: ldaps://ldap.example.com:636
    binddn: cn=account,dc=example,dc=com
    bindpassword: s3cReT_p4s5w0rD
    basedn: dc=example,dc=com
    users_ou: ou=users,dc=example,dc=com
    groups_ou: ou=groups,dc=example,dc=com
    
    # MANDATORY : Name of DN attribute for Users, UserPasswords and Groups
    # You have to set up values for the three, even if you don't use some of the types
    dnAttributes:
      Users: uid
      UserPasswords: uid
      Groups: cn
    
    propagateUserDNChangeOnGroupMember: true
    groupsObjectclass: groupOfNames

    # It is possible to set a default value for some attributes for Users,
    # UserPasswords and Groups. The default value will be set on added and modified
    # events if the local attribute has no value
    defaultValues:
      # Hack to allow creation of an empty group, because of the "MUST member" in schema
      Groups:
        member: ""

    # The local attributes listed here won't be stored in LDAP for Users,
    # UserPasswords and Groups
    attributesToIgnore:
      Users:
        - user_pkey
      UserPasswords:
        - user_pkey
      Groups:
        - group_pkey

hermes-client:
  # If true, will try to remediate errors by merging added and modified events data
  # in error queue when possible
  enableAutoremediation: true

  datamodel:
    Users:
      hermesType: SRVUsers
      # Facultative template of object's string representation that will be used in logs
      toString: "<Users[{{ user_pkey }}, {{ uid | default('#UNDEF#') }}]>"
      attrsmapping:
        user_pkey: srv_user_id
        uid: srv_login
        givenname: srv_firstname
        sn: srv_lastname
        mail: srv_mail
        # Compose the displayname with two other attributes
        displayname: "{{ srv_firstname ~ ' ' ~  srv_lastname }}"
        #
        # Static values
        # Defining them here instead of in default values will allow changes
        # propagation on each entry
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
        # Static values
        # Defining them here instead of in default values will allow changes
        # propagation on each entry
        #
        objectclass: "{{ ['groupOfNames'] }}"

    GroupsMembers:
      hermesType: SRVGroupsMembers
      attrsmapping:
        # 'user_pkey' and 'group_pkey' keys can't be renamed
        user_pkey: srv_user_id
        group_pkey: srv_group_id
```

## Attributes flow

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
  ORA_USERS_FIRSTNAME -->|'title' Jinja filter| SRVUsers_srv_firstname
  ORA_USERS_LASTNAME -->|'title' Jinja filter| SRVUsers_srv_lastname
  ORA_USERS_EMAIL --> SRVUsers_srv_mail

  subgraph SRVUserPasswords
    direction LR
    SRVUserPasswords_srv_user_id["srv_user_id"]
    SRVUserPasswords_srv_password_ldap["srv_password_ldap"]
  end
  ORA_USERPASSWORDS_USER_ID --> SRVUserPasswords_srv_user_id
  ORA_USERPASSWORDS_PASSWORD_ENCRYPTED -->|"'crypto_RSA_OAEP<br /> | ldapPasswordHash'<br />Jinja filter"| SRVUserPasswords_srv_password_ldap
  ORA_USERPASSWORDS_LDAP_HASHES <-->|LDAP_HASHED is filled by,<br /> or provide its value| SRVUserPasswords_srv_password_ldap

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
  ClientGroupsMembers_user_pkey -->|converted to user DN| LDAPGroups_member
  ClientGroupsMembers_group_pkey -->|converted to group DN| LDAPGroups_member

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
