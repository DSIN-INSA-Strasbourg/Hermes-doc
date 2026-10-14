---
title: regex_replace
---

## Description

This plugin allows performing a replacement in a string using the specified regular expression.

## Configuration

Nothing to configure for the plugin.

```yaml
hermes:
  plugins:
    attributes:
      regex_replace:
```

## Usage

```python
regex_replace(value: str, pattern: str, replacement: str, ignorecase: bool = False, multiline: bool = False, count: int = 0, mandatory_count: int = 0) → str
```

As this plugin is just an adaptation of Ansible's `regex_replace_filter`, you can also check [its documentation](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/regex_replace_filter.html).

```yaml
# Will be 'able'
regex_result: "{{ 'ansible' | regex_replace('^a.*i(.*)$', 'a\\1') }}"

# Will be '#CAR\n#tar\nfoo\n#bar\n'
regex_result: "{{ 'CAR\ntar\nfoo\nbar\n' | regex_replace('^(.ar)$', '#\\1', multiline=True, ignorecase=True) }}"


# It is possible to apply the method to a list of strings. Let's take LDAP_PASSWORD_HASHES, which contains a list of LDAP password hashes:
# LDAP_PASSWORD_HASHES:
#   - "{SMD5}NGnIxNg+ZqB3XwhQK/jCRDWWpUQYVbwg"
#   - "{SSHA}9u8ZbEbeLPLI2f4isG7YjJsz6sfonjQAfbbadQ=="
#   - "{SSHA256}l0rZ10MhH6jKGogg2qFvCdiNAqkKVH9OuL0R3FgWRrV4mIaYM2cnYQ=="
#   - "{SSHA512}zKR46tmGg0NKq1FdkmLGZCqXqfnApvFRHSTW4H0Sem9zJH66mgZ6/aB/aypGX+dLAI02akd9lZbplX6y0Typzzir8RIKh6cw,"

# Remove the prefix inside the braces indicating the hash type.
# regex_results:
#   - "NGnIxNg+ZqB3XwhQK/jCRDWWpUQYVbwg"
#   - "9u8ZbEbeLPLI2f4isG7YjJsz6sfonjQAfbbadQ=="
#   - "l0rZ10MhH6jKGogg2qFvCdiNAqkKVH9OuL0R3FgWRrV4mIaYM2cnYQ=="
#   - "zKR46tmGg0NKq1FdkmLGZCqXqfnApvFRHSTW4H0Sem9zJH66mgZ6/aB/aypGX+dLAI02akd9lZbplX6y0Typzzir8RIKh6cw,"
regex_results: "{{ LDAP_PASSWORD_HASHES | map('regex_replace', '^{[^}]+}(.*)$', '\\1') | list }}"
```
