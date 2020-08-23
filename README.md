# hypermodern-python

[![Tests](https://github.com/cli99/hypermodern-python/workflows/Tests/badge.svg)](https://github.com/cli99/hypermodern-python/actions?workflow=Tests)

[![Codecov](https://codecov.io/gh/cli99/hypermodern-python/branch/master/graph/badge.svg)](https://codecov.io/gh/cli99/hypermodern-python)

https://github.com/cjolowicz/hypermodern-python

https://cjolowicz.github.io/posts/hypermodern-python-01-setup/

## Use Poetry to manage Black, Flake8, and the other tools as development dependencies

```
poetry add --dev \
    black \
    flake8 \
    flake8-bandit \
    flake8-black \
    flake8-bugbear \
    flake8-import-order \
    safety
```

## Use pre-commit

```
pip install --user --upgrade pre-commit
```

## Use xdoctest to run documentation exmamples

```
poetry add --dev xdoctest
```

## Create documenation with Sphinx

```
poetry add --dev sphinx
```
