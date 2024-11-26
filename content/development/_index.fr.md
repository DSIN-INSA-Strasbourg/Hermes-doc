---
title: Développement
weight: 5
type: chapter
---


Cette section contient la documentation pour démarrer avec le développement de [plugin](/development/plugins/) et la contribution "noyau" d'Hermes.

## Journalisation

Une instance `Logger` est disponible via la variable "`__hermes__.logger`". Comme cette variable est déclarée comme *builtin*, elle est toujours disponible et ne nécessite aucun `import` ou appel à `logging.getLogger()`.

## Contribuer

Avant de proposer une pull request pour fusionner du code dans Hermes, vous devez vous assurer que votre code :

1. fournit des [docstrings](https://peps.python.org/pep-0257/) et des [annotations de type](https://peps.python.org/pep-0484/)
2. a été formaté avec [black](https://github.com/psf/black)
3. est conforme à [Flake8](https://flake8.pycqa.org)
4. passe la suite de tests

[tox](https://tox.wiki/) peut être utilisé pour valider les trois dernières conditions, en exécutant l'une des commandes ci-dessous :

```bash
# # Test séquentiel (lent mais plus détaillé) uniquement sur la version Python par défaut de votre système
tox run -e linters,tests
# Test en parallèle (plus rapide, mais sans détails) uniquement sur la version Python par défaut de votre système
tox run-parallel -e linters,tests

# Test séquentiel (lent mais plus détaillé) sur toutes les versions Python compatibles - elles doivent être disponibles sur votre système
tox run
# Test en parallèle (plus rapide, mais sans détails) sur toutes les versions Python compatibles - elles doivent être disponibles sur votre système
tox run-parallel
```

{{% notice tip %}}
[tox](https://tox.wiki/) >= 4 doit être installé mais est probablement disponible dans les dépôts de votre distribution
{{% /notice %}}
