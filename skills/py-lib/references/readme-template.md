# Minimal README template

Write the README as if the package already works. Keep it short: a reader should be able to
install, run one example and cite the work without scrolling far. Delete any section marked
optional that does not apply. Fill every `<...>`.

````markdown
# <package-name>

<!-- After the first Zenodo release, paste the DOI badge Markdown that Zenodo shows on the record
     page here (use the concept DOI badge). -->

<One paragraph: what the package computes, for what kind of data, and why someone would use it.
If it accompanies a paper, say so: "This is the reference implementation of the method in
<Authors> (<year>)."

## Installation

```bash
pip install "<package-name> @ git+<repository URL>@v0.1.0"   # a tagged release
# or, once published on PyPI:
pip install <package-name>
uv add <package-name>
```

Requires Python 3.10 or newer.

## Quick start

```python
<5-10 realistic lines: load or generate a small input, call the main function, show the
result. Use the real public API, not print("hello").>
```

More examples are in [`examples/`](examples/).   <!-- optional: only if examples/ exists -->

## Reproducing the results in the paper   <!-- optional: only if the package backs a paper -->

The results in <paper short reference> were produced with version <X.Y.Z>
(tag `v<X.Y.Z>`, archived at doi:<version DOI>). To recreate that environment exactly:

```bash
git clone <repository URL> && cd <project>
git checkout v<X.Y.Z>
uv sync --locked                       # exact versions from uv.lock
uv run python examples/<script>.py     # or the analysis entry point
```

Without uv: `pip install -r requirements-lock.txt && pip install --no-deps .`

## How to cite

If you use this software, please cite <the paper and/or the software>:

> <Family, G.>, <Family, G.> (<year>). <package-name>: <description> (version <X.Y.Z>)
> [Computer software]. doi:<DOI>

```bibtex
@software{<key>,
  author    = {<Family>, <Given> and <Family>, <Given>},
  title     = {<package-name>: <description>},
  year      = {<year>},
  version   = {<X.Y.Z>},
  doi       = {<DOI>},
}
```

Citation metadata is also in [`CITATION.cff`](CITATION.cff); on GitHub, use "Cite this
repository" in the sidebar.

## Development

```bash
uv sync                 # environment with dev tools
uv run pytest           # tests
uv run ruff check .     # lint
uv run ruff format .    # format
```

## License

<SPDX id>. See [`LICENSE`](LICENSE).
````

## Notes

- **Before a DOI exists**, write the "How to cite" section with the repository URL and version
  instead of a DOI, and keep a reminder in the handoff to update it after the first release.
- **Cite the paper, the software, or both?** If the method is the contribution, ask for the paper;
  if people will run the code, also ask for the software version they used. Saying exactly what you
  want cited is the most effective way to get it cited.
- **The BibTeX entry** should agree with `CITATION.cff` (title, authors, version). `@software` is
  supported by biblatex; for plain BibTeX styles that lack it, use `@misc` with
  `howpublished = {Zenodo}` and the same fields.
- **Installing from a tag** (`git+<URL>@v0.1.0`) works before anything is on PyPI and pins the
  exact code.
