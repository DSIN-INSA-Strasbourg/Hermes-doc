---
title: Development
weight: 5
archetype: chapter
---

This section contains the documentation to get started with [plugin](./plugins/) development and Hermes "core" contribution.

## Logging

A `Logger` instance is available through the variable "`__hermes__.logger`". As this var is declared as *builtin*, it is always available and doesn't require any `import` or call to `logging.getLogger()`.

## Contributing

Before submitting a pull request to merge some code in Hermes, you should ensure that:

1. it provides [docstrings](https://peps.python.org/pep-0257/) and [type hints](https://peps.python.org/pep-0484/)
2. it has been formatted with [black](https://github.com/psf/black)
3. it is compliant with [Flake8](https://flake8.pycqa.org)
4. your code doesn't break the test suite

[tox](https://tox.wiki/) may be used to validate the last three conditions, by running one of the commands below :

```bash
# Testing sequentially (slow but more verbose) only on default python version available on your system
tox run -e linters,tests
# Testing in parallel (faster, but without details) only on default python version available on your system
tox run-parallel -e linters,tests

# Testing sequentially (slow but more verbose) on all compatible python versions - they must be available on your system
tox run
# Testing in parallel (faster, but without details) on all compatible python versions - they must be available on your system
tox run-parallel
```

{{% notice tip %}}
[tox](https://tox.wiki/) >= 4 must be installed but is probably available in your distribution's repositories
{{% /notice %}}
