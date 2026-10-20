# The `research/` folder (shared convention)

Every skill in the Aurora Lab research pipeline works on its own. When several are used on the same
project, they cooperate through plain files in one folder instead of calling each other. Nothing here
is required: if a file is missing, the skill asks for what it needs or works from what you give it.

```
research/
  results/      one JSON file per result, e.g. results/ablation.json
  figures/      final figures (PDF, SVG or PNG) and, next to each, <figure>.md with its caption and spec
  claims.md     the claims the paper makes, one per line, each pointing at its evidence
  paper/        the manuscript, one file per section (Markdown or LaTeX)
  refs.bib      bibliography
```

## `results/*.json`

```json
{
  "metric": "AUROC",
  "value": 0.91,
  "ci": [0.88, 0.94],
  "n": 500,
  "unit": null,
  "source": "scripts/eval.py@a1b2c3d",
  "note": "5 seeds, 95% bootstrap interval"
}
```
A file may hold one result object or a list of them. `ci` and `n` are optional but every skill that
writes about results treats a missing interval as "no uncertainty reported", not as "no uncertainty".

## `claims.md`

```
C1 | Our method separates the two populations (AUROC 0.91, 95% CI 0.88–0.94). | results/main.json | checked
C2 | Removing the prior lowers accuracy. | results/ablation.json | open
```
Columns: id, sentence, evidence path(s), status (`open`, `checked`, `revised`, `dropped`).

## Rules every skill follows
- Read whatever already exists; write only your own outputs; never delete another skill's files.
- Never require another skill. Mention a companion skill in prose at most ("pairs well with ...").
- Paths are relative to the project root. If there is no `research/` folder, work where the user points.
