# wipac-dev-py-build-action

GitHub Action to Build a Python Package Using setuptools-scm Version Injection

## Overview

This GHA package validates your project's `pyproject.toml` for correct [`setuptools-scm`](https://pypi.org/project/setuptools-scm/) configuration, then builds the Python package using an injected semantic version.

It is designed to be used in automated release workflows, typically following [`WIPACrepo/wipac-dev-next-version-action`](https://github.com/WIPACrepo/wipac-dev-next-version-action).

## How it Works

1. Ensures your `pyproject.toml`:
    - Lists `setuptools-scm` as a build requirement
    - Does **not** statically define `project.version`
    - Includes the `[tool.setuptools_scm]` section
2. Injects the provided version via `SETUPTOOLS_SCM_PRETEND_VERSION`
3. Builds the package via [`python -m build`](https://pypi.org/project/build/)
    - Both `sdist` and `wheel` artifacts are created
    - **The resulting `.tar.gz` and `.whl` files are written to the `dist/` directory**

## Inputs

| Name      | Required | Description                                                                             |
|-----------|----------|-----------------------------------------------------------------------------------------|
| `version` | `true`   | The semantic version to inject into setuptools-scm via `SETUPTOOLS_SCM_PRETEND_VERSION` |

## Outputs

_None_

## Example Usage

```yaml
jobs:
  ...

  release:
    runs-on: ubuntu-latest
    concurrency: release
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # required to see tags and commits

      - uses: WIPACrepo/wipac-dev-next-version-action@v#.#
        id: next-version
        ...

      - if: steps.next-version.outputs.version != ''
        uses: WIPACrepo/wipac-dev-py-build-action@v#.#
        with:
          version: ${{ steps.next-version.outputs.version }}

      ...
```
