# py-lib

Turns research code into an installable, citable Python package. Start from an idea or from loose
scripts. The agent lays out a `src/` package with `uv`, `hatchling`, `ruff` and `pytest`, and adds
what makes research software citable and reproducible: `CITATION.cff`, a README with "How to
cite", a licence, and a committed `uv.lock`. It copies existing scripts into the package without
rewriting them and deletes nothing. Anything risky (code that runs on import, absolute paths,
undeclared imports) is flagged for you to decide on, not fixed silently.

Optional: GitHub Actions CI, a Dockerfile, an `examples/` folder, a `notebooks/` folder, and a
release checklist for getting a DOI through Zenodo's GitHub integration.

Part of the Aurora Lab research pipeline. Works on its own; pairs well with research-software
(correctness, tests, determinism of the code itself).

## Requirements

- Python 3.10+ and uv (to create the environment and lock file). Without uv the agent still writes
  every file and gives you the commands to run later.
- The first `uv sync` downloads packages from PyPI. The skill itself makes no network calls.
  GitHub, Zenodo and PyPI steps are instructions for you to carry out yourself.
- No bundled scripts. The generated project needs only its own dev tools (pytest, pytest-cov,
  ruff, pyyaml), which uv installs into the project's `.venv`.

## Example prompts

- "Package `fit.py` and `io_utils.py` into a library I can cite in my paper."
- "New Python project for a spike-sorting method: scaffold it with CI and an examples folder."
- "Add a CITATION.cff and a How to cite section to this repo."
- "How do I get a DOI for this code? Prepare the release."
- "Pin the dependencies so reviewers can reproduce Figure 3."

## Files

Writes, in your project: `pyproject.toml`, `uv.lock` (via uv), `.python-version`, `.gitignore`,
`src/<name>/`, `tests/` (including a test that keeps `CITATION.cff` valid and in step with the
version), `README.md`, `CITATION.cff`, `LICENSE`, and on request `examples/`, `notebooks/`,
`.github/workflows/ci.yml`, `Dockerfile`, `.dockerignore`, `requirements-lock.txt`.

Shared `research/` folder (optional, see `references/research-folder.md`): reads
`research/refs.bib` and `research/paper/` to fill the paper citation; appends a software entry
to `research/refs.bib` only if you ask.
