# Canonical `pyproject.toml` template

Fill in the angle-bracket placeholders. Remove the annotation comments before writing the file.

```toml
[build-system]
requires = ["hatchling>=1.27"]
build-backend = "hatchling.build"

[project]
name = "<package-name>"           # distribution name, kebab-case, e.g. "decay-fit"
version = "0.1.0"                 # the ONE place the version is written (see __init__.py)
description = "<one-sentence description>"
readme = "README.md"
license = "<SPDX id>"             # e.g. "MIT", "BSD-3-Clause", "Apache-2.0"; same as LICENSE and CITATION.cff
license-files = ["LICENSE"]
authors = [
  { name = "<Given Family>", email = "<optional email>" },  # same people as CITATION.cff; email may be omitted
]
requires-python = ">=3.10"
keywords = ["<field>", "<method>"]          # optional; helps people find it
classifiers = [                             # optional
  "Intended Audience :: Science/Research",
  "Programming Language :: Python :: 3",
]
dependencies = []                 # run-time deps with lower bounds, e.g. ["numpy>=1.24", "scipy>=1.10"]

# Optional: turn a script's main() into a command, e.g. `decay-fit data.csv`
# [project.scripts]
# <command-name> = "<import_name>.<module>:main"

# Only if a repository exists:
# [project.urls]
# Repository = "<repository URL>"
# Documentation = "<docs URL>"

# Development tools. `uv sync` installs the "dev" group by default; it is never
# part of what users of the package install.
[dependency-groups]
dev = [
  "pytest>=8.0",
  "pytest-cov>=5.0",
  "ruff>=0.6",
  "pyyaml>=6.0",                  # used by tests/test_citation.py
]
# Optional, if the project has notebooks (installed with `uv sync --group notebooks`):
# notebooks = ["jupyterlab>=4"]

# Tell hatch where the package lives (the src/ layout)
[tool.hatch.build.targets.wheel]
packages = ["src/<import_name>"]  # import_name is snake_case, e.g. "decay_fit"

# Optional: keep large or exploratory material out of the source distribution
# [tool.hatch.build.targets.sdist]
# exclude = ["notebooks/", "examples/data/", "examples/output/"]

[tool.ruff]
line-length = 88
target-version = "py310"          # the lowest Python in requires-python

[tool.ruff.lint]
select = [
  "E",   # pycodestyle errors
  "W",   # pycodestyle warnings
  "F",   # pyflakes (unused imports, undefined names)
  "I",   # isort (import ordering)
  "UP",  # pyupgrade (modern syntax for the target version)
]
ignore = []

[tool.ruff.lint.isort]
known-first-party = ["<import_name>"]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--tb=short"
```

## Notes on key choices

**`hatchling` as build backend**
Hatchling understands the `src/` layout through `[tool.hatch.build.targets.wheel]` without
`find_packages()` or path tricks, and `uv build` uses it directly. Version 1.27 or newer is needed
for the SPDX `license` string and `license-files`.

**`src/` layout**
With the package under `src/`, running `pytest` from the project root imports the installed
package, not the raw files, so a missing dependency or a missing `__init__.py` shows up in your
tests instead of in a reader's install.

**Dependencies: lower bounds here, exact versions in `uv.lock`**
`dependencies` lists only what users of the package must install, with the oldest version you
know works (`numpy>=1.24`). Exact pins (`numpy==2.1.3`) in a library make it impossible to install
next to anything else. The exact versions that produced your results are recorded in `uv.lock`
instead; see `reproducibility.md`. Add a dependency with `uv add <pkg>`, which updates both.

**`[dependency-groups]` for dev tools**
Dependency groups (PEP 735) are local to the project: `uv sync` installs `dev` by default, and they
never appear in the published metadata. Optional *features* that users can request
(`pip install decay-fit[plot]`) go in `[project.optional-dependencies]` instead.

**Version in one place**
`version` is written only here. `__init__.py` reads it back with
`importlib.metadata.version("<package-name>")`, and `tests/test_citation.py` checks that
`CITATION.cff` carries the same value. Recent uv versions can bump it for you
(`uv version --bump minor`); otherwise edit this line.

**`requires-python = ">=3.10"`**
A floor, not a pin. Raise it only if the code uses newer syntax or a dependency needs it. The
interpreter you ran on is recorded in `.python-version` and `uv.lock`.

**Ruff `"I"` and `"UP"`**
Import sorting and syntax modernisation in one tool. `target-version` must match the floor in
`requires-python`, or ruff will suggest syntax older interpreters cannot run.

**`pytest-cov`**
Allows `uv run pytest --cov=<import_name> --cov-report=term-missing`. Drop it if the user does not
want coverage reports.
