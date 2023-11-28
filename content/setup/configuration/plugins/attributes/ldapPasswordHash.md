---
title: ldapPasswordHash
---

## Description

This plugin allow to generate LDAP hashes of specified formats from a clear text password string.

## Configuration

You can set up a facultative list of default hash types in plugin settings. This list will be used if hashtypes are not specified in filter arguments, otherwise the specified hashtypes will be used.

```yaml
hermes:
  plugins:
    attributes:
      ldapPasswordHash:
        settings:
          default_hash_types:
            - SMD5
            - SSHA
            - SSHA256
            - SSHA512
```

Valid values for `default_hash_types` are :

- MD5
- SHA
- SMD5
- SSHA
- SSHA256
- SSHA512

## Usage

```python
ldapPasswordHash(password: str, hashtypes: None | str | list[str] = None) → list[str]
```

Once everything is set up, you can generate your hash list  `ldapPasswordHash` key like this  in a Jinja filter :

```yaml
# Will contain a list of hashes of PASSWORD_CLEAR according to
# default_hash_types settings : SMD5, SSHA, SSHA256, SSHA512
ldap_password_hashes: "{{ PASSWORD_CLEAR | ldapPasswordHash }}"

# Will contain a list with only the SSHA512 hashe of PASSWORD_CLEAR
ldap_password_hashes: "{{ PASSWORD_CLEAR | ldapPasswordHash('SSHA512') }}"

# Will contain a list with only the SSHA256 and SSHA512 hashes of PASSWORD_CLEAR
ldap_password_hashes: "{{ PASSWORD_CLEAR | ldapPasswordHash(['SSHA256', 'SSHA512']) }}"
```
