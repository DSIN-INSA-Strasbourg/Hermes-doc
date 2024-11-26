---
title: ldapPasswordHash
---

## Description

Ce plugin permet de générer des hachages LDAP aux formats spécifiés, depuis une chaîne contenant un mot de passe en clair.

## Configuration

Vous pouvez configurer une liste facultative de types de hachage par défaut dans les paramètres du plug-in. Cette liste sera utilisée si les types de hachage ne sont pas spécifiés dans les arguments du filtre, sinon les types de hachage spécifiés seront utilisés.

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

Les valeurs valides pour `default_hash_types` sont :

- MD5
- SHA
- SMD5
- SSHA
- SSHA256
- SSHA512

## Utilisation

```python
ldapPasswordHash(password: str, hashtypes: None | str | list[str] = None) → list[str]
```

Une fois que tout est configuré, vous pouvez générer votre liste de hachages comme ceci dans un filtre Jinja :

```yaml
# Contiendra une liste de hachages de PASSWORD_CLEAR selon les paramètres
# de default_hash_types : SMD5, SSHA, SSHA256, SSHA512
ldap_password_hashes: "{{ PASSWORD_CLEAR | ldapPasswordHash }}"

# Contiendra une liste contenant uniquement le hachage SSHA512 de PASSWORD_CLEAR
ldap_password_hashes: "{{ PASSWORD_CLEAR | ldapPasswordHash('SSHA512') }}"

# Contiendra une liste contenant uniquement les hachages SSHA256
# et SSHA512 de PASSWORD_CLEAR
ldap_password_hashes: "{{ PASSWORD_CLEAR | ldapPasswordHash(['SSHA256', 'SSHA512']) }}"
```
