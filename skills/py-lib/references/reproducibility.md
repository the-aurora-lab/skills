# Recording dependencies for reproducibility

Goal: someone (including you, in two years) can recreate the software environment that produced a
result. This file covers the packaging side only: what is installed, at which versions, on which
Python. Seeds, determinism, data provenance and tests that check the science are research-code
doctrine; the companion skill research-software pairs well with this one for those.

## Two layers, two jobs

| File | Says | Who uses it |
|---|---|---|
| `pyproject.toml` `dependencies` | what the package needs, as lower bounds (`numpy>=1.24`) | anyone installing the package next to their own code |
| `uv.lock` | the exact version (and hash) of every package, direct and indirect, for every supported platform and Python | anyone reproducing *your* results |

Never put exact pins in `dependencies` to "make it reproducible": it breaks installs for everyone
who also needs a different numpy. Reproducibility lives in the lock file.

## Daily workflow

```bash
uv add scipy              # adds "scipy>=<current>" to pyproject.toml and updates uv.lock
uv remove scipy           # the reverse
uv lock                   # re-resolve after editing pyproject.toml by hand
uv lock --upgrade         # move to newer versions on purpose (then rerun tests)
uv sync --locked          # install exactly what uv.lock says; fails if the lock is stale
```

Commit `uv.lock` together with `pyproject.toml`. A change to the lock file in a diff is a change
to the environment and deserves the same review as a code change.

## For people who do not use uv

Export the lock to a pip-readable file and commit it next to `uv.lock`:

```bash
uv export --no-dev --no-emit-project --format requirements-txt -o requirements-lock.txt
```

Readers then run `pip install -r requirements-lock.txt` followed by `pip install --no-deps .` in a
fresh virtual environment. Regenerate the export whenever `uv.lock` changes (add it to the release
checklist). Include `--no-dev` because readers reproducing results do not need pytest or ruff.

Conda users can create an environment with the Python version from `.python-version` and run the
same two pip commands inside it.

## The Python version

- `requires-python` in `pyproject.toml`: the oldest Python the package supports (a floor).
- `.python-version`: the interpreter you developed and ran the results on. uv uses it by default.
- `uv.lock` records the `requires-python` range it was resolved for.

Test the oldest and newest supported versions in CI (see `ci-template.md`).

## Record the environment of a specific run

When a script produces numbers that go into a paper, store a small record next to its outputs.
Stdlib only; put it in the analysis script or an example, not in the library:

```python
import json
import platform
import sys
from importlib.metadata import distributions, version


def environment_record(package: str) -> dict:
    """Versions of Python, the OS, this package and every installed distribution."""
    return {
        "package": package,
        "package_version": version(package),
        "python": sys.version.split()[0],
        "platform": platform.platform(),
        "installed": sorted(f"{d.metadata['Name']}=={d.version}" for d in distributions()),
    }


# e.g. at the end of an analysis script:
# with open("output/environment.json", "w") as fh:
#     json.dump(environment_record("<package-name>"), fh, indent=2)
```

Equivalent one-off from the shell: `uv pip freeze > output/environment.txt`.

## Tie results to a version

Before generating final numbers: commit, bump the version, tag the release (see
`release-and-doi.md`), and run the analysis from that tag with `uv sync --locked`. Then the paper
can say "version 0.3.0 (doi:...)" and the lock file at that tag is the full environment.

## Docker (optional)

A container adds the operating system and system libraries to what is recorded. It is worth it
when the code depends on compiled system libraries or a specific OS; for pure-Python scientific
code, `uv.lock` plus the Python version is usually enough. See `docker-template.md`.
