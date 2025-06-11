---
title: regex_search
---

## Description

Ce plugin permet d'effectuer une recherche dans une chaîne pour en extraire la partie qui correspond à l'expression régulière spécifiée.

## Configuration

Rien à configurer pour le plugin.

```yaml
hermes:
  plugins:
    attributes:
      regex_search:
```

## Utilisation

```python
regex_search(string: str, regex: str, multiline=False, ignorecase=False) → list[str] | None
```

Comme ce plugin n'est qu'une adaptation du filtre `regex_search_filter` d'Ansible, vous pouvez également consulter [sa documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/regex_search_filter.html).

```yaml
# Sera sans valeur (contiendra None)
regex_results: "{{ 'foo' | regex_search('bar') }}"

# Sera sans valeur (contiendra None)
regex_results: "{{ 'foobar' | regex_search('foo$') }}"

# Contiendra 'foo'
regex_results: "{{ 'foobar' | regex_search('^foo') }}"

# Contiendra 'foobar'
regex_results: "{{ 'foobar' | regex_search('^foo.*$') }}"


# Ci-dessous, une approche plus complexe, où LDAP_PASSWORD_HASHES est une liste de hachages de mots de passe LDAP :
# LDAP_PASSWORD_HASHES:
#   - "{SMD5}NGnIxNg+ZqB3XwhQK/jCRDWWpUQYVbwg"
#   - "{SSHA}9u8ZbEbeLPLI2f4isG7YjJsz6sfonjQAfbbadQ=="
#   - "{SSHA256}l0rZ10MhH6jKGogg2qFvCdiNAqkKVH9OuL0R3FgWRrV4mIaYM2cnYQ=="
#   - "{SSHA512}zKR46tmGg0NKq1FdkmLGZCqXqfnApvFRHSTW4H0Sem9zJH66mgZ6/aB/aypGX+dLAI02akd9lZbplX6y0Typzzir8RIKh6cw,"

# Contiendra ['{SSHA}9u8ZbEbeLPLI2f4isG7YjJsz6sfonjQAfbbadQ==']
regex_results: "{{ LDAP_PASSWORD_HASHES | map('regex_search', '^{SSHA}.*$') | reject('none') | list }}"

# Contiendra ['{SSHA}9u8ZbEbeLPLI2f4isG7YjJsz6sfonjQAfbbadQ==', '{SSHA512}zKR46tmGg0NKq1FdkmLGZCqXqfnApvFRHSTW4H0Sem9zJH66mgZ6/aB/aypGX+dLAI02akd9lZbplX6y0Typzzir8RIKh6cw,']
regex_results: "{{ LDAP_PASSWORD_HASHES | map('regex_search', '^({SSHA}|{SSHA512}).*$') | reject('none') | list }}"
```
