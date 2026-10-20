# `CITATION.cff` template and citation test

`CITATION.cff` is a small YAML file in the repository root in the Citation File Format (CFF,
version 1.2.0). GitHub reads it to show a "Cite this repository" box, Zenodo's GitHub integration
reads it for the archive's metadata (unless the repository also has a `.zenodo.json`, which then
replaces it), and reference managers can import it.

## Template

Fill in the placeholders. Delete optional blocks you do not need; keep the commented `doi` line
until the first Zenodo release exists.

```yaml
cff-version: 1.2.0
message: "If you use this software, please cite it using the metadata from this file."
type: software
title: "<package-name>: <one-sentence description>"
version: "0.1.0"
date-released: "<YYYY-MM-DD>"
license: <SPDX id>
authors:
  - family-names: "<Family>"
    given-names: "<Given>"
    affiliation: "<Institution>"          # optional
    orcid: "<full ORCID URL>"             # optional; must be the full URL form
  - family-names: "<Family>"
    given-names: "<Given>"
# repository-code: "<repository URL>"     # once the repository exists
# doi: 10.5281/zenodo.<concept-record-id> # after the first Zenodo release (concept DOI)
keywords:
  - "<field>"
  - "<method>"
abstract: >-
  <Two or three sentences: what the software computes and for which kind of data.>

# Only if there is a paper people should cite together with (or instead of) the software.
preferred-citation:
  type: article
  title: "<Paper title>"
  authors:
    - family-names: "<Family>"
      given-names: "<Given>"
  journal: "<Journal>"                    # for a preprint: type: article, journal: "<server name>"
  year: <YYYY>
  # doi: <10.xxxx/yyyy>                   # once the paper has one
```

## Field notes

- **Required fields** (the file is invalid without them): `cff-version`, `message`, `title`,
  `authors`. Each author needs `family-names` + `given-names`, or a single `name` for a
  consortium or organisation.
- **Quote `version` and `date-released`.** Unquoted, `version: 1.0` is read as the number 1.0 and
  `version: 1.10` as 1.1. Dates must be `YYYY-MM-DD`.
- **`version` must match `pyproject.toml`.** The test below enforces it. Update `version` and
  `date-released` in the same commit that bumps the package version.
- **`doi`**: use the Zenodo *concept* DOI (it always resolves to the newest version). The version
  DOI of a release cannot be in that release's own `CITATION.cff`, because Zenodo mints it after
  the release is published. See `release-and-doi.md`.
- **`orcid`**: the full URL form (the orcid.org address followed by the 16-digit iD), not just the
  digits.
- **`license`**: an SPDX identifier, identical to `pyproject.toml` and the `LICENSE` file.
- **`preferred-citation`**: when present, GitHub's citation box shows the paper instead of the
  software. If you want both cited, say so in the README's "How to cite" section.
- **`references`** (optional list): software or papers this code builds on, in the same format as
  `preferred-citation`. Useful for crediting a method you re-implemented.

Optional extra validation against the official schema, run by the user (downloads the validator
from PyPI the first time): `uvx cffconvert --validate`. The pytest check below covers the common
mistakes without any network access.

## `tests/test_citation.py`

Replace `<package-name>` with the distribution name. Needs `pyyaml` in the dev group.

```python
"""Keep CITATION.cff valid and in step with the package version."""

from importlib.metadata import version
from pathlib import Path

import yaml

CFF = Path(__file__).resolve().parents[1] / "CITATION.cff"
REQUIRED = {"cff-version", "message", "title", "authors"}


def load_cff() -> dict:
    data = yaml.safe_load(CFF.read_text(encoding="utf-8"))
    assert isinstance(data, dict), "CITATION.cff must be a YAML mapping"
    return data


def test_citation_has_required_fields():
    cff = load_cff()
    missing = REQUIRED - cff.keys()
    assert not missing, f"CITATION.cff is missing {sorted(missing)}"
    assert cff["authors"], "CITATION.cff needs at least one author"
    for author in cff["authors"]:
        assert author.get("family-names") or author.get("name"), author


def test_citation_version_matches_package():
    cff_version = load_cff().get("version")
    assert isinstance(cff_version, str), "quote the version in CITATION.cff"
    assert cff_version == version("<package-name>")


def _strings(node):
    if isinstance(node, dict):
        for value in node.values():
            yield from _strings(value)
    elif isinstance(node, list):
        for value in node:
            yield from _strings(value)
    elif isinstance(node, str):
        yield node


def test_citation_has_no_placeholders():
    left = [s for s in _strings(load_cff()) if "<" in s and ">" in s]
    assert not left, f"fill in the placeholders in CITATION.cff: {left}"
```

The third test looks only at parsed values, so commented-out lines (the future `doi`) do not
trigger it. It fails while placeholders remain, which is the point: it keeps an unfinished citation
from being released. If the user wants to release with a known gap, they can mark that test
`@pytest.mark.xfail` deliberately rather than delete it.
