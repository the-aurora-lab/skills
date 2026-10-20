# `examples/` and `notebooks/` convention

Use only when the user asked for one or both folders.

```
examples/
  README.md            one line per example: what it shows and roughly how long it runs
  01_<topic>.py        short script that runs top to bottom
  02_<topic>.py
  data/                tiny input files only (well under a few MB in total)
  output/              written by the examples; in .gitignore
notebooks/             optional
  01_<topic>.ipynb     tutorial or exploration; committed WITHOUT outputs
```

## Examples

- Each example imports the installed package (`from <import_name> import ...`). Never use
  `sys.path.append` or relative paths into `src/`.
- Resolve files relative to the script, never absolute paths:
  ```python
  from pathlib import Path

  HERE = Path(__file__).resolve().parent
  data = HERE / "data" / "example.csv"
  out = HERE / "output"
  out.mkdir(exist_ok=True)
  ```
- Prefer generating synthetic input in the script (with a fixed seed) over shipping data files.
  If real data is needed and large, archive it separately (e.g. its own Zenodo record) and tell the
  reader in `examples/README.md` how to fetch it; do not download at import time.
- Keep each example fast (seconds). An example that takes an hour belongs in the paper's analysis
  code, not here.
- Numbered prefixes give a reading order. Run one with `uv run python examples/01_<topic>.py`.
- If an example plots, save the figure to `output/` and do not call `plt.show()`, so it also runs
  headless and in tests.
- Examples are where a script's former top-level code (loading, fitting, plotting) belongs when it
  is moved out of a package module.

## Smoke test for the examples

Add `tests/test_examples.py` so a broken example fails CI:

```python
"""Every script in examples/ must run to completion."""

import runpy
from pathlib import Path

import pytest

EXAMPLES = sorted((Path(__file__).resolve().parents[1] / "examples").glob("*.py"))


@pytest.mark.parametrize("script", EXAMPLES, ids=lambda p: p.name)
def test_example_runs(script, monkeypatch):
    monkeypatch.setenv("MPLBACKEND", "Agg")  # no windows during tests
    runpy.run_path(str(script), run_name="__main__")
```

If some examples are slow, mark them with a custom `slow` marker and deselect by default.

## Notebooks

- Commit notebooks without outputs; outputs bloat the repository and make diffs unreadable. Clear
  them before committing: `uv run jupyter nbconvert --clear-output --inplace notebooks/*.ipynb`
  (or add `nbstripout` to the dev group and run `uv run nbstripout --install` once to do it as a
  git filter).
- Put Jupyter in its own dependency group so the default environment stays lean:
  ```toml
  [dependency-groups]
  notebooks = ["jupyterlab>=4"]
  ```
  and start it with `uv sync --group notebooks && uv run jupyter lab`.
- Notebooks call the package; they do not define functions the package needs. If a notebook grows
  a useful function, move it into `src/` and test it.
- Ruff lints `.ipynb` files too; that is fine and catches unused imports in notebooks.

## Packaging

Neither folder goes into the wheel: hatch only packages `src/<import_name>`. The source
distribution includes them by default; exclude bulky material with
`[tool.hatch.build.targets.sdist] exclude = ["notebooks/", "examples/data/", "examples/output/"]`.
