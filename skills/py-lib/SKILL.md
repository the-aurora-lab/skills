---
name: py-lib
description: Turns research code into an installable, citable Python package (uv, hatchling, ruff, pytest, src/ layout) with CITATION.cff, a README with "How to cite", a lock file for reproducibility, and optional CI, Docker and examples/notebooks folders. Use when someone wants to "package my research code", "make this installable", "turn these scripts into a library", "add a CITATION.cff", "get a DOI for my code", or starts a new Python project for a paper.
---

# py-lib

You turn research code into a Python package that other people can install, run and cite. The
toolchain is fixed: `uv` for environments and the lock file, `hatchling` as the build backend,
`ruff` for lint and format, `pytest` for tests, and a `src/` layout so tests never import the
un-installed source by accident. On top of that every package gets the three things that make
research software citable and reproducible: a `CITATION.cff`, a README with a "How to cite"
section, and a committed `uv.lock`.

The user starts either from an idea (nothing exists yet) or from existing scripts. Both paths end
at the same package.

Scope: this skill builds the package structure and its metadata. It does not judge whether the
science in the code is correct, how good the tests are, or how to make runs deterministic; for that
doctrine, the companion skill research-software pairs well with this one. Do not rewrite the
user's code here.

---

## Opening a session

Before writing a single file, read the room.

**Step 1: Is there existing code?** If the user points at a directory or files, read them first
(Path B). If nothing exists yet, this is Path A.

**Step 2: Path A (raw idea).** Ask at most three questions:
1. What does the package do, in one sentence?
2. What would a realistic first call look like? e.g. `params = mylib.fit_decay(t, y)`. This shapes
   the public API and the README quick start.
3. What should it be called? (Distribution names are `kebab-case`, import names `snake_case`, same
   root: `decay-fit` -> `import decay_fit`.)

**Step 3: Ask about metadata and extras once, in one short message**, not mid-scaffold:
- Authors for the citation: names, and ORCID iDs and affiliations if they have them.
- Licence (default MIT; BSD-3-Clause and Apache-2.0 are common alternatives in research).
- Is there a paper (published, preprint, or in preparation) that people should cite with the code?
- Extras: GitHub Actions CI? A Dockerfile? An `examples/` folder? A `notebooks/` folder?

If the user skips any of these, proceed with placeholders and list them in the handoff. If a
`research/` folder exists, read `research/refs.bib` and the title page in `research/paper/` for
the paper's title and authors before asking (see "Shared research folder" below).

**Goal before writing any file:** you can state what the package does, its top-level public API,
its name, and who the authors are (or that they are placeholders).

---

## Two starting paths

### Path A: raw idea

Follow the full scaffold sequence. Generate stub implementations (functions that raise
`NotImplementedError`, with a docstring) so the project installs and its tests run before any real
logic exists.

### Path B: existing research code

The user has loose scripts, a flat module, or a half-built package. Before scaffolding:

1. **Read every file.** Note what is a reusable function, what is a script entry point, what is
   shared utility code, and which third-party packages are imported.
2. **Pick the layout.** Usually one package with one module per script; split into sub-packages
   only if the code already has that shape.
3. **Propose the mapping explicitly and wait for a yes**, e.g. "I'll copy `fit_decay.py` and
   `load_csv.py` into `src/decay_fit/`, keep both files' contents as they are, and re-export
   `fit_decay` and `load_series` from `__init__.py`." Copy, never delete: the originals stay where
   they are until the user removes them.
4. **Preserve existing logic verbatim.** Relocate and re-expose; do not refactor, rename, or
   reformat unless asked. This is the highest-risk moment: the code produced someone's results.
5. **Name the unavoidable edits in the proposal.** The only change you make without being asked
   is to imports between the user's own scripts, which break once they live in a package
   (`from load_csv import load_series` becomes `from .load_csv import load_series`). Say so in the
   mapping.
6. **Flag, do not fix** (list these in the proposal and let the user choose):
   - top-level code that runs on import (loading data, fitting, plotting, `plt.show()`) outside an
     `if __name__ == "__main__":` block: offer to move it under that guard or into `examples/`;
   - absolute or machine-specific paths (a home directory, a drive letter, a cluster scratch path): offer to make them arguments;
   - `sys.path` manipulation, `from x import *`, or modules that shadow a third-party name;
   - imports of packages that are not installed or not obvious from the name (`sklearn` ->
     `scikit-learn`, `cv2` -> `opencv-python`, `yaml` -> `pyyaml`).
7. A script with a `main()` that parses arguments can become a command: offer a
   `[project.scripts]` entry (see the pyproject template).
8. Then scaffold everything the code lacks.

---

## Scaffold sequence

Generate all the files below, then give a concise summary. Do not stop for permission between files.

### Directory structure

```
<project-root>/
├── src/
│   └── <import_name>/
│       ├── __init__.py
│       └── <module>.py
├── tests/
│   ├── __init__.py
│   ├── test_<module>.py
│   └── test_citation.py
├── pyproject.toml
├── uv.lock                 (generated by uv, never hand-written)
├── .python-version
├── .gitignore
├── README.md
├── CITATION.cff
├── LICENSE
├── examples/               (if requested)
├── notebooks/              (if requested)
├── .github/workflows/ci.yml  (if CI requested)
├── Dockerfile              (if Docker requested)
└── .dockerignore           (if Docker requested)
```

The `src/` layout prevents Python from importing the un-installed package from the repo root,
which would hide missing dependencies and missing `__init__.py` files.

### `pyproject.toml`

Read `references/pyproject-template.md` and fill it in:
- `name`: distribution name (kebab-case); `version`: `"0.1.0"`; `description`: one sentence.
- `authors`: the same people as in `CITATION.cff`.
- `license`: an SPDX identifier matching `LICENSE` and `CITATION.cff`.
- `requires-python`: `">=3.10"` unless the code needs newer syntax or the user says otherwise.
- `dependencies`: only what the package imports at run time, with lower bounds, never exact pins.
  Exact versions belong in `uv.lock` (see reproducibility below).
- Dev tools (pytest, pytest-cov, ruff, pyyaml for the citation test) go in
  `[dependency-groups] dev`, which `uv sync` installs by default.
- `[project.urls]`: only if the user has a repository; otherwise omit the table.

Ruff configuration lives in `pyproject.toml` (no separate `.ruff.toml`): line length 88, rules
`E`, `W`, `F`, `I`, `UP` at minimum.

### `.python-version`

One line with the Python minor version the user develops and runs their results on, e.g. `3.12`
(ask, or use the interpreter they already have; `uv python pin <version>` writes this file). It
must satisfy `requires-python`. This records the interpreter for reproducibility; it does not
restrict who can install the package.

### `.gitignore`

At least: `.venv/`, `__pycache__/`, `*.pyc`, `dist/`, `build/`, `*.egg-info/`, `.pytest_cache/`,
`.ruff_cache/`, `.coverage`, `.ipynb_checkpoints/`, `examples/output/`. Do not ignore `uv.lock`.

### `src/<import_name>/__init__.py`

Single-source the version from the installed metadata so it is written in one place
(`pyproject.toml`) and cannot drift:

```python
"""<One-sentence description of the package>."""

from importlib.metadata import version

__version__ = version("<package-name>")
```

For Path B, also re-export the public functions and list them in `__all__`.

### `src/<import_name>/<module>.py`

Path A: a stub of the first function or class implied by the described API, with a docstring and
`raise NotImplementedError`. Path B: the relocated code, unchanged except for the import fix named
in the mapping.

### `tests/`

- `tests/__init__.py`: empty.
- `tests/test_<module>.py`: one test that imports the package and checks `__version__` is a
  string; one test per public function. Path A: assert the stub raises `NotImplementedError` (a
  failing-until-implemented test is better than none). Path B: call each function on a tiny
  synthetic input whose answer is known (e.g. data generated from known parameters, a three-row
  CSV written to `tmp_path`) and assert the result. This is a smoke test that the package works
  after the move, not a validation of the science.
- `tests/test_citation.py`: from `references/citation-template.md`. It fails if `CITATION.cff` is
  not valid YAML, lacks a required field, or its `version` differs from the package's.

### `CITATION.cff` and `LICENSE`

Read `references/citation-template.md`. Fill in title, version, date, licence and authors; add
`preferred-citation` only if there is a paper; leave the `doi` line commented out until the first
Zenodo release. Write `LICENSE` with the full licence text, the current year and the copyright
holder the user names (default: the authors).

### `README.md`

Read `references/readme-template.md`. Write it as if the package already works. It must contain:
one-paragraph description, installation, a realistic quick start, "Reproducing the results" (only
if the package backs a paper), "How to cite", development commands, and the licence.

### Optional: `examples/` and `notebooks/`

If requested, read `references/examples-and-notebooks.md` and follow its convention: examples are
short scripts that import the installed package and run top to bottom; notebooks are committed
without outputs; neither ships in the wheel. Add the examples smoke test from that file.

### Optional: `.github/workflows/ci.yml`

Read `references/ci-template.md`. The workflow runs on push and PR to `main`, tests the oldest
supported and the newest Python, installs from the lock file, and runs ruff before pytest.

### Optional: `Dockerfile` and `.dockerignore`

Read `references/docker-template.md`. Multi-stage build that installs exactly the locked versions.

### Create the lock file and check the scaffold

If `uv` is available and the user agrees (the first sync downloads packages from PyPI), run from
the project root:

```bash
uv sync                    # creates .venv and uv.lock
uv run ruff check .
uv run ruff format --check .
uv run pytest
```

Report what passed. If ruff flags code that was moved verbatim (Path B), report the findings and
ask before running `ruff check --fix` or `ruff format`: both rewrite the user's code. If `uv` is not
available, list these commands in the handoff instead and say that `uv.lock` does not exist yet.

---

## Reproducibility: record what the results ran on

Read `references/reproducibility.md` when the package backs published results, when the user asks
how to "pin" or "freeze" dependencies, or when someone else must rerun the analysis. The short
version:
- `pyproject.toml` says what the package needs (lower bounds); `uv.lock` records the exact versions
  that were resolved. Commit both. Reproduce with `uv sync --locked`.
- For people who do not use uv, export the lock: `uv export --no-dev --no-emit-project
  --format requirements-txt -o requirements-lock.txt`.
- Tag the release that produced a paper's numbers and cite that exact version.

## Releases, versions and a DOI

Read `references/release-and-doi.md` when the user wants a DOI, a release, a version bump, or a
"Code availability" statement. These are steps the user performs on GitHub and Zenodo in a
browser; you prepare the files (version bump, `CITATION.cff`, README badge placeholder, release
notes) and give them the checklist. Never call GitHub, Zenodo or PyPI yourself.

---

## Handoff

After the scaffold, give the user the commands, not a wall of text:

```bash
uv sync                        # environment + lock file
uv run pytest                  # tests, including the CITATION.cff check
uv run ruff check .            # lint
uv run ruff format .           # format
uv build                       # wheel + sdist in dist/
```

Then say briefly what is not done yet:
- stubs still raise `NotImplementedError` (Path A);
- placeholders left in `CITATION.cff`, `LICENSE` or `README.md` (list them);
- no DOI yet: point to the release checklist in `references/release-and-doi.md`;
- PyPI publishing is not configured (optional; a Trusted Publisher is the usual route when ready);
- creating the GitHub repository is the user's step.

On Path B, also list anything you flagged and did not fix (top-level code, absolute paths,
undeclared imports, ruff findings in moved code), and the original files left in place for the
user to delete once they are happy with the package.

---

## Updating an existing scaffold

When the user comes back to add one thing (CI, a dependency, a citation file, a DOI), make a
targeted edit: add the dependency to the existing list with `uv add <pkg>` or by editing
`dependencies`, then `uv lock`; emit only the new files. Do not regenerate files that already exist.

---

## Shared research folder

This skill works on its own. If the project has a `research/` folder (convention in
`references/research-folder.md`), it may:
- read `research/refs.bib` and `research/paper/` to fill `preferred-citation` in `CITATION.cff`
  and the "How to cite" section;
- append a BibTeX entry for the software to `research/refs.bib` only if the user asks, never
  editing or deleting existing entries.

It writes nothing else under `research/`. Once a release is tagged, results files can name the
exact code in their `source` field (e.g. `decay-fit==0.2.0` or the tag).

---

## Reference files

- `references/pyproject-template.md`: annotated `pyproject.toml`. Read when generating the file.
- `references/citation-template.md`: `CITATION.cff` template, field notes, and the citation test.
  Read when writing `CITATION.cff`.
- `references/readme-template.md`: minimal README with "How to cite". Read when writing README.md.
- `references/reproducibility.md`: lock file, exports, recording a run's environment. Read when
  results must be reproducible or the user asks about pinning.
- `references/release-and-doi.md`: versioning, release checklist, Zenodo DOI steps, code
  availability statement. Read when the user wants a release or a DOI.
- `references/examples-and-notebooks.md`: `examples/` and `notebooks/` convention and smoke test.
  Read only if requested.
- `references/ci-template.md`: GitHub Actions workflow. Read only if CI is requested.
- `references/docker-template.md`: Dockerfile and `.dockerignore`. Read only if Docker is requested.
- `references/research-folder.md`: the shared `research/` folder convention.
