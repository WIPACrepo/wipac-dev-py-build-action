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

The following is based on `WIPACrepo/wipac-dev-actions-testbed`'s [`cicd.yml`](https://github.com/WIPACrepo/wipac-dev-actions-testbed/blob/main/.github/workflows/cicd.yml):

```yaml
jobs:

  ...

  tag-and-release:
  # only run on main/master/default
  if: format('refs/heads/{0}', github.event.repository.default_branch) == github.ref
  needs: [
    ...
  ]
  runs-on: ubuntu-latest
  concurrency: release  # prevent any possible race conditions
  steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0  # required to see tags and commits
        ref: ${{ github.sha }}  # lock to triggered commit ('github.ref' is dynamic)

    - uses: actions/setup-python@v5  # needed for building project
      with:
        python-version: "${{ fromJSON(needs.py-versions.outputs.matrix)[0] }}"

    - uses: WIPACrepo/wipac-dev-next-version-action@v1.1
      id: next-version
      ...

    - if: steps.next-version.outputs.version != ''
      name: Tag New Version
      ...

    - if: steps.next-version.outputs.version != ''
      uses: WIPACrepo/wipac-dev-py-build-action@v1.0
      # -> uses the most recent git tag for versioning
      # -> creates 'dist/' files

    - if: steps.next-version.outputs.version != ''
      uses: softprops/action-gh-release@v2
      with:
        files: dist/*
        tag_name: v${{ steps.next-version.outputs.version }}  # must match git tag above
        generate_release_notes: true
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    - if: steps.next-version.outputs.version != ''
      uses: pypa/gh-action-pypi-publish@release/v1
      with:
        user: __token__
        password: ${{ secrets.PYPI_TOKEN }}

```
