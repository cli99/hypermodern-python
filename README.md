# hypermodern-python-cli99

[![Tests](https://github.com/cli99/hypermodern-python-cli99/workflows/Tests/badge.svg)](https://github.com/cli99/hypermodern-python-cli99/actions?workflow=Tests)
[![Codecov](https://codecov.io/gh/cli99/hypermodern-python-cli99/branch/master/graph/badge.svg)](https://codecov.io/gh/cli99/hypermodern-python-cli99)
[![PyPI](https://img.shields.io/pypi/v/hypermodern-python-cli99.svg)](https://pypi.org/project/hypermodern-python-cli99/)
[![Read the Docs](https://readthedocs.org/projects/hypermodern-python-cli99/badge/)](https://hypermodern-python-cli99.readthedocs.io/)

This repo follows [hypermodern-python](https://github.com/cjolowicz/hypermodern-python).

> **_NOTE:_**

### Installing python with pyenv

Install pyenv

```
curl https://pyenv.run | bash

```

Add the following into the ~/.bashrc or ~/.zshrc

```
export PATH="~/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Install python build dependencies

```
sudo apt update && sudo apt install -y make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev \
libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl git
```

Install python 3.7.7 and make it available in the project

```
pyenv install 3.7.7
pyenv local 3.7.7
```

## Setup

### Setting up a python project using Poetry

Install poetry

```
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
source ~/.poetry/env
```

Initialize a python project

```
poetry init --no-interaction
```

Modify the configuration file [pyproject.toml](pyproject.toml).

## Testing

Install pytest

```
poetry add --dev pytest
```

Organize tests in a separate file hierarchy next to src, named tests

```
├── src
└── tests
    ├── __init__.py
    └── test_xxx.py
```

The file **init**.py is empty and serves to declare the test suite as a package. The file test_xxx.py contains a test case for the xxx module.

### Code coverage with Coverage.py

```
poetry add --dev coverage[toml] pytest-cov
```

Config Coverage.py using the pyproject.toml configuration file, provided it was installed with the toml extra as shown above.
To enable coverage reporting, invoke pytest with the --cov option:

```
poetry run pytest --cov
```

### Test automation with Nox

```
pip install --user --upgrade nox
```

Nox gives you a quick and informative overview of the developer tasks it automates

```
 nox --list-sessions
```

Nox uses a standard Python file, noxfile.py, for configuration.

Nox creates virtual environments for the listed Python versions (3.8 and 3.7), and runs the session inside each environment. You can speed things up by passing the --reuse-existing-virtualenvs (-r) option:

```
nox -r
```

Run a specific test module inside the environments:

```
nox -- tests/test_xxx.py
```

### Mocking with pytest-mock

```
poetry add --dev pytest-mock

```

### End-to-end testing

Use Pytest’s markers to apply a custom mark. This will allow you to select or skip them later, using Pytest’s `-m` option.

```python
# tests/test_xxx.py
@pytest.mark.e2e
def test_main_succeeds_in_production_env(runner):
    result = runner.invoke(xxx.main)
    assert result.exit_code == 0
```

Exclude end-to-end tests from automated testing by passing `-m "not e2e"` to Pytest:

```python
# noxfile.py
import nox


@nox.session(python=["3.8", "3.7"])
def tests(session):
    args = session.posargs or ["--cov", "-m", "not e2e"]
    session.run("poetry", "install", external=True)
    session.run("pytest", *args)
```

Run end-to-end tests by passing `-m e2e` to the Nox session, using a double dash (--) to separate them from Nox’s own options. For example, here’s how you would run end-to-end tests inside the testing environment for Python 3.8:

```
nox -rs tests-3.8 -- -m e2e

```

## Linting

Use Poetry to manage Black, Flake8, and the other tools as development dependencies

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

The function install_with_constraints below is a wrapper for session.install. It generates a constraints file by running poetry export, and passes that file to pip using its `--constraint` option. The function uses the standard tempfile module to create a temporary file for the constraints.

```python
# noxfile.py
def install_with_constraints(session, *args, **kwargs):
    with tempfile.NamedTemporaryFile() as requirements:
        session.run(
            "poetry",
            "export",
            "--dev",
            "--format=requirements.txt",
            f"--output={requirements.name}",
            external=True,
        )
        session.install(f"--constraint={requirements.name}", *args, **kwargs)
```

### Managing Git hooks with pre-commit

```yml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.3.0
    hooks:
      - id: check-yaml
      - id: end-of-file-fixer
      - id: trailing-whitespace
  - repo: local
    hooks:
      - id: black
        name: black
        entry: poetry run black
        language: system
        types: [python]
      - id: flake8
        name: flake8
        entry: poetry run flake8
        language: system
        types: [python]
```

```

pip install --user --upgrade pre-commit

```

The hooks run automatically every time you invoke git commit, applying checks to any newly created or modified files. When you add new hooks, you can trigger them manually for all files using the following command:

```
pre-commit run --all-files
```

## [Typing](https://cjolowicz.github.io/posts/hypermodern-python-04-typing/)

## Documentation

### Linting code documentation with flake8-docstrings

The flake8-docstrings plugin uses the tool pydocstyle to check that docstrings are compliant with the style recommendations of PEP 257. Warnings range from missing docstrings to issues with whitespace, quoting, and docstring content.

```
poetry add --dev flake8-docstrings

```

### Adding docstrings to Nox sessions

### Validating docstrings against function signatures with darglint

Darglint checks that docstring descriptions match function definitions, and integrates with Flake8 as a plugin.

```
poetry add --dev darglint
```

Configure darglint to accept one-line docstrings, using the `.darglint` configuration file:

```
# .darglint
[darglint]
strictness = short
```

### Use xdoctest to run documentation exmamples

```

poetry add --dev xdoctest

```

### Create documenation with Sphinx

```

poetry add --dev sphinx

```

Create a directory docs. The master document is located in the file `docs/index.rst`

Create the Sphinx configuration file docs/conf.py. This provides meta information about your project:

```python
# docs/conf.py
"""Sphinx configuration."""
project = "hypermodern-python"
author = "Your Name"
copyright = f"2020, {author}"
```

```
nox -rs docs
```

The generated documentation is at `docs/_build/index.html`.

### Generating API documentation with autodoc

- autodoc enables Sphinx to generate API documentation from the docstrings in your package.
- napoleon pre-processes Google-style docstrings to reStructuredText.
- sphinx-autodoc-typehints uses type annotations to document the types of function parameters and return values.

The autodoc and napoleon extensions are distributed with Sphinx.

```
poetry add --dev sphinx-autodoc-typehints
```

Install the extension and your own package into the Nox session. Your package is needed so Sphinx can read its documentation strings and type annotations.

```python
# noxfile.py
@nox.session(python="3.8")
def docs(session: Session) -> None:
    """Build the documentation."""
    session.run("poetry", "install", "--no-dev", external=True)
    install_with_constraints(session, "sphinx", "sphinx-autodoc-typehints")
    session.run("sphinx-build", "docs", "docs/_build")
```

Activate the extensions by declaring them in the Sphinx configuration file:

```python
# docs/conf.py
project = "hypermodern-python"
author = "Your Name"
copyright = f"2020, {author}"
extensions = [
    "sphinx.ext.autodoc",
    "sphinx.ext.napoleon",
    "sphinx_autodoc_typehints",
]
```

## CI/CD

### Tests

```yml
# .github/workflows/tests.yml
name: Tests
on: push
jobs:
  tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.7', '3.8']
    name: Python ${{ matrix.python-version }}
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v1
        with:
          python-version: ${{ matrix.python-version }}
          architecture: x64
      - run: pip install nox==2019.11.9
      - run: pip install poetry==1.0.5
      - run: nox
```

Add the line below to the top of your README.md to display the badge:

```
[![Tests](https://github.com/<your-username>/hypermodern-python/workflows/Tests/badge.svg)](https://github.com/<your-username>/hypermodern-python/actions?workflow=Tests)
```

### Coverage reporting with Codecov

```
poetry add --dev codecov
```

Add the Nox session shown below. The session exports the coverage data to cobertura XML format, which is the format expected by Codecov. It then uses codecov to upload the coverage data.

```python
# noxfile.py
@nox.session(python="3.8")
def coverage(session: Session) -> None:
    """Upload coverage data."""
    install_with_constraints(session, "coverage[toml]", "codecov")
    session.run("coverage", "xml", "--fail-under=0")
    session.run("codecov", *session.posargs)
```

```yml
# .github/workflows/coverage.yml
name: Coverage
on: push
jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v1
        with:
          python-version: '3.8'
          architecture: x64
      - run: pip install nox==2019.11.9
      - run: pip install poetry==1.0.5
      - run: nox --sessions tests-3.8 coverage
        env:
          CODECOV_TOKEN: ${{secrets.CODECOV_TOKEN}}
```

Add the Codecov badge to your README.md:

```
[![Codecov](https://codecov.io/gh/<your-username>/hypermodern-python/branch/master/graph/badge.svg)](https://codecov.io/gh/<your-username>/hypermodern-python)
```

### Uploading your package to PyPI

```
poetry build
```

```
poetry publish
```

or

```yml
# .github/workflows/release.yml
name: Release
on:
  release:
    types: [published]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v1
        with:
          python-version: '3.8'
          architecture: x64
      - run: pip install nox==2019.11.9
      - run: pip install poetry==1.0.5
      - run: nox
      - run: poetry build
      - run: poetry publish --username=__token__ --password=${{ secrets.PYPI_TOKEN }}
```

Add a badge to README.md which links to your PyPI project page and displays the latest release:

```
[![PyPI](https://img.shields.io/pypi/v/hypermodern-python.svg)](https://pypi.org/project/hypermodern-python/)
```

### Documenting releases with Release Drafter

The Release Drafter action drafts your next release notes as pull requests are merged into master. It does this by creating and maintaining a draft release. When a pull request gets accepted, the release description is updated to include its title, author, and a link to the pull request itself.
When you’re ready to make a release, you simply need to add the tag name, and click Publish.

```yml
# .github/workflows/release-drafter.yml
name: Release Drafter
on:
  push:
    branches:
      - master
jobs:
  draft_release:
    runs-on: ubuntu-latest
    steps:
      - uses: release-drafter/release-drafter@v5.6.1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The configuration file below goes somewhat further, and is very loosely based on the Angular Commit Message Convention and gitmoji. You will need to add the remaining labels to your repository manually, using New Label on the Issues tab of the repository.

```yml
# .github/release-drafter.yml
categories:
  - title: ':boom: Breaking Changes'
    label: 'breaking'
  - title: ':package: Build System'
    label: 'build'
  - title: ':construction_worker: Continuous Integration'
    label: 'ci'
  - title: ':books: Documentation'
    label: 'documentation'
  - title: ':rocket: Features'
    label: 'enhancement'
  - title: ':beetle: Fixes'
    label: 'bug'
  - title: ':racehorse: Performance'
    label: 'performance'
  - title: ':hammer: Refactoring'
    label: 'refactoring'
  - title: ':fire: Removals and Deprecations'
    label: 'removal'
  - title: ':lipstick: Style'
    label: 'style'
  - title: ':rotating_light: Testing'
    label: 'testing'
template: |
  ## What’s Changed

  $CHANGES
```

### Single-sourcing the package version

```
poetry add --python="<3.8" importlib_metadata
```

```python
# src/hypermodern_python/__init__.py
"""The hypermodern Python project."""
try:
    from importlib.metadata import version, PackageNotFoundError  # type: ignore
except ImportError:  # pragma: no cover
    from importlib_metadata import version, PackageNotFoundError  # type: ignore


try:
    __version__ = version(__name__)
except PackageNotFoundError:  # pragma: no cover
    __version__ = "unknown"
```

With this in place, pyproject.toml has become the single source of truth for your package version.

### Uploading your package to TestPyPI

TestPyPI is a separate instance of the Python Package Index that allows you to try distribution tools and processes without affecting the real index.

```
# .github/workflows/test-pypi.yml
name: TestPyPI
on:
  push:
    branches:
      - master
jobs:
  test_pypi:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-python@v1
      with:
        python-version: '3.8'
        architecture: x64
    - run: pip install poetry==1.0.5
    - run: >-
        poetry version patch &&
        version=$(poetry version | awk '{print $2}') &&
        poetry version $version.dev.$(date +%s)
    - run: poetry build
    - uses: pypa/gh-action-pypi-publish@v1.0.0a0
      with:
        user: __token__
        password: ${{ secrets.TEST_PYPI_TOKEN }}
        repository_url: https://test.pypi.org/legacy/
```

### Hosting documentation at Read the Docs

The install section in the configuration file tells Read the Docs how to install your package and its documentation dependencies. Read the Docs does not support installation of dependencies using Poetry directly. pip can install any package with a standard pyproject.toml file, and will use Poetry behind the scenes.

```yml
# .readthedocs.yml
version: 2
sphinx:
  configuration: docs/conf.py
formats: all
python:
  version: 3.7
  install:
    - requirements: docs/requirements.txt
    - path: .
```

```txt
# docs/requirements.txt
sphinx==2.3.1
sphinx-autodoc-typehints==1.10.3
```

gn up at Read the Docs, and import your GitHub repository, using the button Import a Project. Read the Docs automatically starts building your documentation. When the build has completed, your documentation will have a public URL like this:
`https://hypermodern-python.readthedocs.io/`

You can display the documentation link on PyPI by including it in your package configuration file:

```
# pyproject.toml
[tool.poetry]
documentation = "https://hypermodern-python.readthedocs.io"
```

Add the Read the Docs badge to README.md:

```
[![Read the Docs](https://readthedocs.org/projects/hypermodern-python/badge/)](https://hypermodern-python.readthedocs.io/)
```
