# wipac-dev-py-build-action

GitHub Action to Build a Python Package Using `setuptools-scm`

## Overview

This GHA package validates your project's `pyproject.toml` for correct [`setuptools-scm`](https://pypi.org/project/setuptools-scm/) configuration, then builds the Python package.

It is designed to be used in automated release workflows, typically preceding GitHub release and/or PyPI publishing.

## How it Works

1. Ensures your `pyproject.toml`:
    - Lists `setuptools-scm` as a build requirement
    - Does **not** statically define `project.version`
    - Includes the `[tool.setuptools_scm]` section
1. Builds the package via [`python -m build`](https://pypi.org/project/build/)
    - Both `sdist` and `wheel` artifacts are created
    - **The resulting `.tar.gz` and `.whl` files are written to the `dist/` directory**
1. **You must have the minimum supported Python version declared in `pyproject.toml` installed in the environment.**
    - For example, if your project requires `>=3.10`, you must use `actions/setup-python` with Python 3.10 before this step.

## Inputs

_None_

## Outputs

_None_

## Example Usage

The following is based on `WIPACrepo/wipac-dev-actions-testbed`'s [`release.yml`](https://github.com/WIPACrepo/wipac-dev-actions-testbed/blob/main/.github/workflows/release.yml):

```yaml
jobs:

  ...

  py-build:
    needs: [ ... ]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          ref: ${{ github.sha }}  # in case 'ref' (arg default) has been updated since start

      - uses: actions/setup-python@v5
        with:
          python-version: <your project's min version>  # 👈 must match `requires-python`

      - uses: WIPACrepo/wipac-dev-py-build-action@main
        # ^^^ creates dist/

      - name: Upload dist/ artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  github-release:
    needs: [ py-build ]
    runs-on: ubuntu-latest
    steps:
      ...

  pypi-publish:
    needs: [ py-build ]
    runs-on: ubuntu-latest
    steps:
      ...

```
