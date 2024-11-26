---
title: hermes
weight: 1
---

Paramètres partagés par le serveur et tous les clients.

Sous-sections principales :

- hermes
  - [cache](/setup/configuration/hermes/#hermes.cache)
  - [cli_socket](/setup/configuration/hermes/#hermes.cli_socket)
  - [logs](/setup/configuration/hermes/#hermes.logs)
  - [mail](/setup/configuration/hermes/#hermes.mail)
  - [plugins](/setup/configuration/hermes/#hermes.plugins)
    - [attributes](/setup/configuration/hermes/#hermes.plugins.attributes)
    - [datasources](/setup/configuration/hermes/#hermes.plugins.datasources)
    - [messagebus](/setup/configuration/hermes/#hermes.plugins.messagebus)

---

## hermes.umask {#hermes.umask}

- *Description* : Définit le umask par défaut pour chaque fichier ou répertoire créé par l'application : répertoires cache, fichiers cache et fichiers journaux. **Attention** comme c'est une valeur octale, elle doit être préfixée par un `0`.
- *Obligatoire* : Non
- *Type* : integer
- *Valeurs valides* : 0000 - 0777
- *Valeur par défaut* : 0027

## hermes.cache {#hermes.cache}

Section obligatoire pour définir les paramètres du cache.

### hermes.cache.dirpath {#hermes.cache.dirpath}

- *Description* : Chemin d'un répertoire existant où les fichiers de cache seront stockés.
- *Obligatoire* : **Oui**
- *Type* : string

### hermes.cache.enable_compression {#hermes.cache.enable_compression}

- *Description* : Si `true`, tous les fichiers cache seront compressés avec gzip.
- *Obligatoire* : Non
- *Type* : boolean
- *Valeur par défaut* : `true`

### hermes.cache.backup_count {#hermes.cache.backup_count}

- *Description* : À chaque sauvegarde, si le contenu du fichier a changé, Hermes conservera le contenu du cache précédent jusqu'au *backup_count* spécifié.
- *Obligatoire* : Non
- *Type* : integer
- *Valeurs valides* : 0 - 999999
- *Valeur par défaut* : 1

## hermes.cli_socket {#hermes.cli_socket}

Active le socket CLI qui permet la communication entre l'application et sa CLI.

### hermes.cli_socket.path {#hermes.cli_socket.path}

- *Description* : Chemin d'accès au fichier de socket CLI à créer/utiliser. Si ce chemin n'est pas spécifié, la CLI sera désactivée.
- *Obligatoire* : Non
- *Type* : string

### hermes.cli_socket.owner {#hermes.cli_socket.owner}

- *Description* : Nom de l'utilisateur qui sera propriétaire du fichier socket lors de sa création, tel qu'il serait transmis à chown.  
    Si ce nom n'est pas spécifié, le nom de l'utilisateur qui exécute le serveur hermes sera utilisé.
- *Obligatoire* : Non
- *Type* : string
- *Ignoré lorsque* : [dont_manage_sockfile](/setup/configuration/hermes/#hermes.cli_socket.dont_manage_sockfile) vaut `true`

### hermes.cli_socket.group {#hermes.cli_socket.group}

- *Description* : Nom du groupe qui sera propriétaire du fichier socket lors de sa création, tel qu'il serait transmis à chown.  
    Si ce nom n'est pas spécifié, le nom de du groupe qui exécute le serveur hermes sera utilisé.
- *Obligatoire* : Non
- *Type* : string
- *Ignoré lorsque* : [dont_manage_sockfile](/setup/configuration/hermes/#hermes.cli_socket.dont_manage_sockfile) vaut `true`

### hermes.cli_socket.mode {#hermes.cli_socket.mode}

- *Description* : Les autorisations à appliquer au fichier socket lors de sa création, telles qu'elles seraient transmises à chmod.  
    Pour les personnes habituées à /usr/bin/chmod, rappelez-vous que les modes sont des nombres octaux et doivent être préfixés par un `0`.  
    Si le mode n'est pas spécifié et que le fichier socket n'existe pas, le umask par défaut du système sera utilisé lors de la définition du mode pour le fichier socket nouvellement créé.  
    Si le mode n'est pas spécifié et que le fichier socket existe, le mode du fichier socket existant sera utilisé.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 00600
- *Valeurs valides* : 0 - 07777
- *Ignoré lorsque* : [dont_manage_sockfile](/setup/configuration/hermes/#hermes.cli_socket.dont_manage_sockfile) vaut `true`

### hermes.cli_socket.dont_manage_sockfile {#hermes.cli_socket.dont_manage_sockfile}

- *Description* : Indique qu'Hermes ne doit pas gérer la création du fichier socket et utiliser le descripteur de fichier socket fourni par son processus parent (généralement SystemD).  
    Le socket créé doit être un socket stream AF_UNIX d'écoute.  
    Un et un seul socket doit être fourni : Hermes s'en assurera en vérifiant que la variable d'environnement SystemD `LISTEN_FDS` est définie sur `1`, et échouera dans le cas contraire.
- *Obligatoire* : Non
- *Type* : boolean
- *Valeur par défaut* : `false`

## hermes.logs {#hermes.logs}

Section obligatoire pour définir les paramètres des fichiers log.

### hermes.logs.logfile {#hermes.logs.logfile}

- *Description* : Chemin d'un répertoire existant où les fichiers journaux seront stockés. Si ce chemin n'est pas spécifié, aucun fichier journal ne sera stocké sur le disque.
- *Obligatoire* : **Oui**
- *Type* : string

### hermes.logs.backup_count {#hermes.logs.backup_count}

- *Description* : Hermes assurera une rotation de son fichier log tous les jours à minuit et conservera les *backup_count* versions précédentes des fichiers journaux.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 7
- *Valeurs valides* : 0 - 999999

### hermes.logs.verbosity {#hermes.logs.verbosity}

- *Description* : Verbosité de la journalisation.
- *Obligatoire* : Non
- *Type* : string
- *Valeur par défaut* : warning
- *Valeurs valides* :
  - critical
  - error
  - warning
  - info
  - debug

### hermes.logs.long_string_limit {#hermes.logs.long_string_limit}

- *Description* : Définit la limite (taille maximale) du contenu des attributs de chaîne à afficher dans les journaux.  
    Si le contenu d'un attribut de chaîne est supérieur à cette limite, il sera tronqué à cette limite et marqué comme LONG_STRING dans les journaux.  
    Peut être défini à `null` pour désactiver cette fonctionnalité et toujours afficher l'intégralité de la chaîne dans les journaux.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 512
- *Valeurs valides* : [1 - 999999] ou `null`

## hermes.mail {#hermes.mail}

Section obligatoire pour définir les paramètres de messagerie (e-mail) permettant à Hermes de notifier les erreurs aux administrateurs.

Les e-mails contiendront 3 pièces jointes lorsque cela est possible : `previous.txt`, `current.txt` et `diff.txt`, contenant l'état précédent, l'état actuel et la différence entre les états précédent et actuel.

### hermes.mail.server {#hermes.mail.server}

- *Description* : Nom DNS ou adresse IP du relais SMTP.
- *Obligatoire* : **Oui**
- *Type* : string

### hermes.mail.from {#hermes.mail.from}

- *Description* : Adresse e-mail qui sera définie comme adresse d'expéditeur, avec la syntaxe de messagerie `Nom d'utilisateur <nom@example.com>`
- *Obligatoire* : **Oui**
- *Type* : string

### hermes.mail.to {#hermes.mail.to}

- *Description* : Adresse ou liste d'adresses de destination.
- *Obligatoire* : **Oui**
- *Type* : string | string[]

### hermes.mail.compress_attachments {#hermes.mail.compress_attachments}

- *Description* : Indique si les pièces jointes doivent être compressées ou envoyées brutes.
- *Obligatoire* : Non
- *Type* : boolean
- *Valeur par défaut* : `true`

### hermes.mail.mailtext_maxsize {#hermes.mail.mailtext_maxsize}

- *Description* : Taille maximale en octets pour le contenu de l'e-mail. Si la taille du contenu est supérieure à *mailtext_maxsize*, un message de substitution par défaut sera envoyé à la place.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 1048576 *(1 Mo)*
- *Valeurs valides* : >= 0

### hermes.mail.attachment_maxsize {#hermes.mail.attachment_maxsize}

- *Description* : Taille maximale en octets pour une seule pièce jointe de l'e-mail. Si la taille de la pièce jointe est supérieure à *attachment_maxsize*, elle ne sera pas jointe à l'e-mail et un message l'indiquant sera ajouté au contenu de l'e-mail.
- *Obligatoire* : Non
- *Type* : integer
- *Valeur par défaut* : 5242880 *(5 Mo)*
- *Valeurs valides* : >= 0

## hermes.plugins {#hermes.plugins}

Section obligatoire pour déclarer quels plugins doivent être chargés, avec leurs paramètres.

Elle est divisée en sous-sections par type de plugin.

### hermes.plugins.attributes {#hermes.plugins.attributes}

Section facultative pour déclarer les plugins d'attributs à charger, ainsi que leurs paramètres.

Elle doit contenir une sous-section nommée avec le nom du plugin contenant elle-même une sous-section facultative `settings` avec les paramètres du plugin, à renseigner selon la [documentation du plugin](/setup/configuration/plugins/attributes/).

Exemple avec le plugin `ldapPasswordHash` :

```yaml
hermes:
  # (...)
  plugins:
    attributes:
      ldapPasswordHash:
        settings:
          default_hash_types:
            - SMD5
            - SSHA
            - SSHA256
            - SSHA512
  # (...)
```

### hermes.plugins.datasources {#hermes.plugins.datasources}

Section obligatoire pour `hermes-server` pour déclarer la ou les sources de données et leurs paramètres. Si elle est définie sur un `hermes-client`, elle sera ignorée silencieusement.

Un même plugin de source de données peut être utilisé pour interroger plusieurs sources de données, donc pour chaque source de données souhaitée, vous devez déclarer une sous-section avec le nom souhaité de la source de données (qui sera utilisé dans le modèle de données), contenant deux entrées obligatoires :

- `type` (*string*) : le plugin de source de données à utiliser pour cette source de données.
- `settings` (*subsection*) : les paramètres du plugin de source de données pour cette source de données selon la [documentation du plugin](/setup/configuration/plugins/datasources/).

Exemple :

```yaml
hermes:
  # (...)
  plugins:
    datasources:
      my_oracle1_datasource:
        type: oracle
        settings:
          login: HERMES_DUMMY
          password: "DuMmY_p4s5w0rD"
          port: 1234
          server: dummy.example.com
          sid: DUMMY
      
      my_oracle2_datasource:
        type: oracle
        settings:
          login: HERMES_DUMMY2
          password: "DuMmY2_p4s5w0rD"
          port: 1234
          server: dummy.example.com
          sid: DUMMY2

      my_ldap_datasource:
        type: ldap
        settings:
          uri: ldaps://dummy.example.com:636
          binddn: cn=binddn,dc=example,dc=com
          bindpassword: DuMmY_p4s5w0rD
          basedn: dc=example,dc=com
  # (...)
```

### hermes.plugins.messagebus {#hermes.plugins.messagebus}

Section obligatoire pour déclarer le plugin de bus de messages à charger, ainsi que ses paramètres. Évidemment, vous devez configurer exactement un plugin de bus de messages.

- Sur `hermes-server`, il recherchera le *plugin producteur de bus de messages* dans le répertoire `plugins/messagebus_producers/`.
- Sur `hermes-client`, il recherchera le *plugin consommateur de bus de messages* dans le répertoire `plugins/messagebus_consumers/`.

Elle doit contenir une sous-section nommée avec le nom du plugin contenant une sous-section facultative `settings` avec les paramètres du plugin à définir selon la documentation du plugin [producteur de bus de messages](/setup/configuration/plugins/messagebus_producers/) ou [consommateur de bus de messages](/setup/configuration/plugins/messagebus_consumers/).

Exemple avec le plugin producteur `sqlite` :

```yaml
hermes:
  # (...)
  plugins:
    messagebus:
      sqlite:
        settings:
          uri: /path/to/hermes/sqlite/message/bus.sqlite
          retention_in_days: 30
  # (...)
```
