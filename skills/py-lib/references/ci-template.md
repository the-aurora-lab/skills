# GitHub Actions CI workflow template

Path: `.github/workflows/ci.yml`

Fill in `<import_name>` (snake_case, e.g. `decay_fit`) and the two Python versions: the floor from
`requires-python` and the newest version your dependencies support.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.10", "3.14"]

    steps:
      - uses: actions/checkout@v7

      - name: Install uv and select Python ${{ matrix.python-version }}
        uses: astral-sh/setup-uv@v10.3.0   # exact release tag: setup-uv has no @v10 tag
        with:
          python-version: ${{ matrix.python-version }}   # sets UV_PYTHON for later steps

      - name: Install from the lock file
        run: uv sync --locked

      - name: Lint with ruff
        run: uv run ruff check .

      - name: Check formatting
        run: uv run ruff format --check .

      - name: Run tests
        run: uv run pytest --cov=<import_name> --cov-report=term-missing
```

## Notes on key choices

**`python-version` from the matrix**
`.python-version` pins the interpreter uv uses by default. The action's `python-version` input sets
`UV_PYTHON`, which overrides that file for the rest of the job; without it every matrix job would
quietly run on the one pinned version, and the matrix would test nothing. uv downloads the requested
Python if the runner lacks it.

**Action versions**
From v8 on, `astral-sh/setup-uv` publishes only immutable full-version tags (`@v10.3.0`), not
moving major tags such as `@v10`, so the full tag is required. For the strongest supply-chain
guarantee, pin the commit SHA the action's README shows, with the version in a comment. These were
the current releases in October 2026; check the actions' release pages when you set up CI and
let Dependabot or Renovate keep them current. The uv cache is on by default on GitHub-hosted
runners (`enable-cache: auto`).

**`uv sync --locked`**
Installs exactly the versions in `uv.lock` (plus the `dev` group) and fails if the lock is out of
date with `pyproject.toml`. That catches "forgot to commit the lock" before a reader does. The lock
is universal, so the same file serves every Python in the matrix.

**Oldest and newest Python**
Testing the floor of `requires-python` and the newest supported version catches most compatibility
breaks without multiplying CI time.

**Ruff before pytest**
Lint and format failures show separately from test failures.

**`--cov-report=term-missing`**
Prints coverage, including uncovered lines, in the job log. No upload step needed.

**Pinning uv itself (optional)**
For a fully repeatable CI, add `version: "<the uv version you use locally>"` under `with:` in the
setup-uv step; `uv --version` prints it.

**No publish step**
Publishing to PyPI is a separate workflow the user adds when ready (Trusted Publisher via OIDC,
`pypa/gh-action-pypi-publish`). Zenodo archiving needs no workflow: it is triggered by publishing a
GitHub release (see `release-and-doi.md`).
