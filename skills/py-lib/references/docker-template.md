# Docker template

Two files to emit when the user requests Docker support. Fill in `<X.Y>` with the version in
`.python-version` (e.g. `3.12`), and `<command-name>` if the package defines a
`[project.scripts]` command.

---

## `Dockerfile`

Multi-stage build: the build stage installs exactly the versions in `uv.lock` into a virtual
environment; the runtime stage carries only that environment.

```dockerfile
# syntax=docker/dockerfile:1

# -- Build stage --------------------------------------------------------------
FROM python:<X.Y>-slim AS builder

# Pin uv for repeatable builds: replace "latest" with your `uv --version`, e.g. 0.11.7
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    UV_PROJECT_ENVIRONMENT=/opt/venv

WORKDIR /build

# Dependencies first (cached layer unless pyproject.toml or uv.lock change)
COPY pyproject.toml uv.lock ./
RUN uv sync --locked --no-dev --no-install-project

# Then the package itself (hatchling needs README.md and LICENSE to build)
COPY README.md LICENSE ./
COPY src/ src/
RUN uv sync --locked --no-dev --no-editable

# -- Runtime stage ------------------------------------------------------------
FROM python:<X.Y>-slim AS runtime

COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

WORKDIR /work

# For a package with a command, make it the entry point; for a pure library, the
# default is a Python prompt with the package importable.
# ENTRYPOINT ["<command-name>"]
CMD ["python"]
```

---

## `.dockerignore`

```
.git
.github
.venv
__pycache__
*.pyc
*.egg-info
dist/
build/
.pytest_cache/
.ruff_cache/
.ipynb_checkpoints/
notebooks/
examples/output/
.DS_Store
```

---

## Notes

**Locked install**
`uv sync --locked --no-dev` installs the exact versions from `uv.lock`, without pytest or ruff, and
fails if the lock is stale. The image therefore matches the environment recorded in the repository.

**Two `uv sync` steps**
The first installs only the dependencies, so editing your code does not reinstall numpy. The second
installs the package itself, non-editable, so the runtime stage does not need `src/`.

**Same base image in both stages**
The virtual environment points at the interpreter in the base image, so both stages must use the
same `python:<X.Y>-slim` tag. `UV_PYTHON_DOWNLOADS=never` makes uv use that interpreter instead of
downloading another.

**Usage**
```bash
docker build -t <package-name>:<version> .
docker run --rm <package-name>:<version> python -c "import <import_name>; print(<import_name>.__version__)"
docker run --rm -v "$PWD/data:/work/data" <package-name>:<version> python my_analysis.py
```

**When Docker is worth it**
Mostly when the code needs system libraries or a specific OS, or runs as a service. For a
pure-Python library that people `pip install` into their own environment, `uv.lock` is usually
enough. For the record of a paper, tag the image with the release version, and pin the base image
by digest (`python:<X.Y>-slim@sha256:...`) if you need the OS layer to be bit-identical later.
