---
title: Development
weight: 5
archetype: chapter
---

This section contains the documentation to get started with [plugin](./plugins/) development and Hermes "core" contribution.

## Logging

A `Logger` instance is available through the variable "`__hermes__.logger`". As this var is declared as *builtin*, it is always available and doesn't require any `import` or call to `logging.getLogger()`.

## Coding style

Before submitting a pull request to merge some code in Hermes, you should ensure that:

- it has been formatted with [black](https://github.com/psf/black)
- it provides [docstrings](https://peps.python.org/pep-0257/) and [type hints](https://peps.python.org/pep-0484/)
