# Versions, releases and a DOI through Zenodo

Everything in this file is done by the user, in a browser or with git. The agent prepares the files
and hands over the checklist; it never calls GitHub, Zenodo or PyPI.

## Versioning

Use `MAJOR.MINOR.PATCH` (semantic versioning):
- `0.y.z` while the API is still moving. Most research packages live here for a long time.
- **PATCH** (`0.3.0` -> `0.3.1`): bug fixes that do not change the public API. Note in the release
  notes if a fix changes numerical results, even slightly; readers of your paper need to know.
- **MINOR** (`0.3.1` -> `0.4.0`): new features; while on `0.y`, also breaking changes.
- **MAJOR** (`1.0.0`): you promise the public API is stable.

Rules that matter for research:
- One tag per state of the code that produced reported results. Never move or delete a published
  tag; release a new version instead.
- Git tags are `v` + the version: `v0.3.0`.
- The version is written in `pyproject.toml` and copied to `CITATION.cff`; `tests/test_citation.py`
  fails if they disagree.

## Release checklist (give this to the user)

```bash
# 1. bump the version (recent uv; otherwise edit `version` in pyproject.toml)
uv version --bump minor
# 2. edit CITATION.cff: version: "<new>", date-released: "<today>"
# 3. refresh the lock and its pip export, then check everything
uv lock
uv export --no-dev --no-emit-project --format requirements-txt -o requirements-lock.txt
uv run ruff check . && uv run pytest
# 4. commit and tag
git add -A && git commit -m "Release v<new>"
git tag -a v<new> -m "v<new>"
git push && git push --tags
```

5. On GitHub: Releases -> "Draft a new release" -> choose tag `v<new>` -> title `v<new>` -> short
   notes (what changed, and whether any results change) -> Publish. **Publishing a GitHub release
   (not just pushing a tag) is what triggers Zenodo.**

## Getting a DOI with Zenodo's GitHub integration (one-time setup)

Zenodo is a free, long-term research archive run by CERN. Its GitHub integration archives a
snapshot of every GitHub release and gives it a DOI.

1. The repository must be public on GitHub: Zenodo can only access public repositories.
2. Sign in at zenodo.org with "Log in with GitHub" (or link GitHub from an existing Zenodo account).
3. In Zenodo, open the account menu -> GitHub. Click the sync button if the repository is not
   listed, then switch the repository's toggle on. (Zenodo's help pages describe the current
   screens if these labels have changed.)
4. Optional dry run: Zenodo runs a separate sandbox site (sandbox.zenodo.org) with its own test
   DOIs. Use it if you want to see what a record will look like first.
5. Publish a GitHub release (checklist step 5). Zenodo then creates a record with a zip of the
   code at that tag; this usually takes minutes, longer for a large release or when Zenodo is busy.
6. Open the new record on Zenodo and check the metadata (title, authors, ORCID iDs, licence,
   description). Zenodo reads `CITATION.cff`, but only a subset of its fields. If the repository
   also has a `.zenodo.json`, Zenodo uses that file alone and ignores `CITATION.cff` entirely, so
   keep one source of metadata: add `.zenodo.json` only for fields `CITATION.cff` cannot carry, and
   then keep both files in step. Fix anything wrong by editing the record on Zenodo, then fix the
   source file so the next release is right.

## Concept DOI vs version DOI

Zenodo mints two kinds of DOI:
- a **version DOI** for each release: points at exactly that snapshot;
- a **concept DOI** for the project: always resolves to the newest version.

Use them like this:
- **Paper (Methods, "Code availability")**: the *version* DOI of the release that produced the
  results, plus the version number.
- **`CITATION.cff` `doi` and the README badge**: the *concept* DOI. A release's own `CITATION.cff`
  cannot contain that release's version DOI, because Zenodo mints it after publication; the concept
  DOI is stable from the first release on.

After the first release: uncomment `doi:` in `CITATION.cff` with the concept DOI, add the badge
Markdown that the Zenodo record page offers to the top of the README, put the DOI into "How to
cite", and include these edits in the next release.

## Code availability statement (template for the paper)

> The software is available at <repository URL> under the <licence> licence. The version used in
> this study (<X.Y.Z>) is archived at Zenodo (doi:<version DOI>). The exact environment is recorded
> in the `uv.lock` file of that release.

## Related, optional

- **Data**: archive large datasets as their own Zenodo record (or a domain repository) and cite
  that DOI separately; do not put them in the package.
- **PyPI**: publishing is independent of the DOI. When ready, configure a Trusted Publisher for
  the repository on PyPI and publish from CI; until then, users can install from a tag
  (`pip install "<name> @ git+<repository URL>@v0.3.0"`).
- **Software Heritage** archives public repositories automatically and gives persistent
  identifiers (SWHIDs) for exact commits; it complements, not replaces, a Zenodo DOI.
