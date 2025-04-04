---
title: hermes
weight: 1
---

Settings shared by server and all clients.

Main subsections:

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

- *Description*: Set up the default umask for each file or directory created by the application : cache dirs, cache files and log files. **Warning** as it is an octal value, it must be prefixed by a `0`.
- *Mandatory*: No
- *Type*: integer
- *Valid values*: 0000 - 0777
- *Default value*: 0027

## hermes.cache {#hermes.cache}

Mandatory section to define cache settings.

### hermes.cache.dirpath {#hermes.cache.dirpath}

- *Description*: Path of an existing directory where cache files will be stored.
- *Mandatory*: **Yes**
- *Type*: string

### hermes.cache.enable_compression {#hermes.cache.enable_compression}

- *Description*: If `true`, all cache files will be gzipped.
- *Mandatory*: No
- *Type*: boolean
- *Default value*: `true`

### hermes.cache.backup_count {#hermes.cache.backup_count}

- *Description*: At each save, if the file content has changed, Hermes will keep previous cache content up to specified *backup_count*.
- *Mandatory*: No
- *Type*: integer
- *Valid values*: 0 - 999999
- *Default value*: 1

## hermes.cli_socket {#hermes.cli_socket}

Enable CLI socket that will allow communication between app and its CLI.

### hermes.cli_socket.path {#hermes.cli_socket.path}

- *Description*: Path to CLI socket file to create/use. When left unspecified, CLI will be disabled.
- *Mandatory*: No
- *Type*: string

### hermes.cli_socket.owner {#hermes.cli_socket.owner}

- *Description*: Name of the user that should own the socket file when created, as would be fed to chown.  
    When left unspecified, it uses the current hermes-server running user.
- *Mandatory*: No
- *Type*: string
- *Ignored when*: [dont_manage_sockfile](/setup/configuration/hermes/#hermes.cli_socket.dont_manage_sockfile) is `true`

### hermes.cli_socket.group {#hermes.cli_socket.group}

- *Description*: Name of the group that should own the socket file when created, as would be fed to chown.  
    When left unspecified, it uses the current group of hermes-server running user.
- *Mandatory*: No
- *Type*: string
- *Ignored when*: [dont_manage_sockfile](/setup/configuration/hermes/#hermes.cli_socket.dont_manage_sockfile) is `true`

### hermes.cli_socket.mode {#hermes.cli_socket.mode}

- *Description*: The permissions to apply to the socket file when created, as would be fed to chmod.  
    For those used to /usr/bin/chmod remember that modes are octal numbers and should be prefixed by a `0`.  
    If mode is not specified and the socket file does not exist, the default umask on the system will be used when setting the mode for the newly created socket file.  
    If mode is not specified and the socket file does exist, the mode of the existing socket file will be used.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 00600
- *Valid values*: 0 - 07777
- *Ignored when*: [dont_manage_sockfile](/setup/configuration/hermes/#hermes.cli_socket.dont_manage_sockfile) is `true`

### hermes.cli_socket.dont_manage_sockfile {#hermes.cli_socket.dont_manage_sockfile}

- *Description*: Indicates that Hermes shouldn't handle the socket file creation, and use the socket file descriptor provided by its parent process (typically SystemD).  
    The created socket must be a listening AF_UNIX stream socket.
    One and only one socket must be provided : Hermes will ensure this by checking that the SystemD env var `LISTEN_FDS` is set to `1`, and will fail otherwise.
- *Mandatory*: No
- *Type*: boolean
- *Default value*: `false`

## hermes.logs {#hermes.logs}

Mandatory section to define log settings.

### hermes.logs.logfile {#hermes.logs.logfile}

- *Description*: Path of an existing directory where log files will be stored. When left unspecified, no log file will be stored on disk.
- *Mandatory*: **Yes**
- *Type*: string

### hermes.logs.backup_count {#hermes.logs.backup_count}

- *Description*: Hermes will rotate its log every day at midnight and keep up to specified *backup_count* values of previous log files.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 7
- *Valid values*: 0 - 999999

### hermes.logs.verbosity {#hermes.logs.verbosity}

- *Description*: Log verbosity.
- *Mandatory*: No
- *Type*: string
- *Default value*: warning
- *Valid values*:
  - critical
  - error
  - warning
  - info
  - debug

### hermes.logs.long_string_limit {#hermes.logs.long_string_limit}

- *Description*: Define the limit (max size) of string attributes content to show in logs.  
   If a string attribute content is greater than this limit, it will be truncated to this limit and marked as a LONG_STRING in logs.  
  Can be set to `null` to disable this feature and always show full string content in logs.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 512
- *Valid values*: [1 - 999999] or `null`

## hermes.mail {#hermes.mail}

Mandatory section to define mail settings to allow Hermes to notify errors to admins.

The email will contain 3 attachments when possible: `previous.txt`, `current.txt`, and `diff.txt`, containing the previous state, the current state, and the diff between previous and current states.

### hermes.mail.server {#hermes.mail.server}

- *Description*: DNS name or IP address of SMTP relay.
- *Mandatory*: **Yes**
- *Type*: string

### hermes.mail.from {#hermes.mail.from}

- *Description*: E-mail address that will be set as from address in the mail syntax `User name <name@example.com>`
- *Mandatory*: **Yes**
- *Type*: string

### hermes.mail.to {#hermes.mail.to}

- *Description*: Recipient address or list of addresses.
- *Mandatory*: **Yes**
- *Type*: string | string[]

### hermes.mail.compress_attachments {#hermes.mail.compress_attachments}

- *Description*: Indicate if attachments must be gzipped or sent raw.
- *Mandatory*: No
- *Type*: boolean
- *Default value*: `true`

### hermes.mail.mailtext_maxsize {#hermes.mail.mailtext_maxsize}

- *Description*: Max size in bytes for mail content. If content size is greater than *mailtext_maxsize*, then a default fallback message will be set instead.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 1048576 *(1 MB)*
- *Valid values*: >= 0

### hermes.mail.attachment_maxsize {#hermes.mail.attachment_maxsize}

- *Description*: Max size in bytes for a single mail attachment. If the attachment size is greater than *attachment_maxsize*, it will not be attached to the email and a message indicating this will be added to the mail content.
- *Mandatory*: No
- *Type*: integer
- *Default value*: 5242880 *(5 MB)*
- *Valid values*: >= 0

## hermes.plugins {#hermes.plugins}

Mandatory section to declare which plugins must be loaded, with their settings.

It is divided into subsections by plugin type.

### hermes.plugins.attributes {#hermes.plugins.attributes}

Facultative section to declare the attributes plugins to load, and their settings.

It must contain a subsection named with the plugin name containing a facultative `settings` subsection with the plugin settings to fill according to the [plugin documentation](/setup/configuration/plugins/attributes/).

Example with the `ldapPasswordHash` plugin:

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

Mandatory section on `hermes-server` to declare the datasource(s), and their settings. If set on `hermes-clients`, it will be silently ignored.

A same datasource plugin can be used for several datasources, so for each datasource needed, you must declare a subsection with your desired datasource name (that will be used in datamodel), containing two mandatory entries:

- `type` (*string*): the datasource plugin to use for this datasource.
- `settings` (*subsection*): the datasource plugin settings for this datasource according to the [plugin documentation](/setup/configuration/plugins/datasources/).

Example:

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

Mandatory section to declare the messagebus plugin to load, and its settings. Obviously, you must set up exactly one message bus plugin.

- On `hermes-server`, it will look up for *Message bus producer plugin* in `plugins/messagebus_producers/` directory.
- On `hermes-client`, it will look up for *Message bus consumer plugin* in `plugins/messagebus_consumers/` directory.

 It must contain a subsection named with the plugin name containing a facultative `settings` subsection with the plugin settings to fill according to the [messagebus producers](/setup/configuration/plugins/messagebus_producers/) or [messagebus consumers](/setup/configuration/plugins/messagebus_consumers/) plugin documentation.

Example with the `sqlite` producer plugin:

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
