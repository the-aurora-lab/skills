# test-readers

Find out how real readers will receive a draft before you send it. Your agent gives the draft, and
nothing else, to blind reader personas (Newcomer, Expert, Editor, Decision-maker, Skimmer) and
compares what each one took away with what you meant to say. You get a tactful report that ranks
every issue by severity and pairs it with a concrete fix or question.

## What is inside

- `SKILL.md`: the method (scoping, persona choice, the blind packet, the persona prompt, the
  comprehension probe, severity ranking, report format and tone).
- `examples/sample-draft.md`: a synthetic one-page grant summary with realistic problems.
- `examples/sample-report.md`: the severity-ranked report a real run produced on that draft.
- `references/research-folder.md`: the shared `research/` folder convention.

## Requirements

Nothing to install: no scripts, no network access. It works best in an agent that can run subagents
or start a fresh session, so each reader meets the draft for the first time. In an agent that cannot,
it reads as each persona in separate passes and says so in the report.

## Files it reads and writes (all optional)

- Reads the draft you point to (Markdown, plain text, LaTeX, or anything your agent can read, such as
  a PDF). With no path and a `research/` folder, it looks in `research/paper/`; it may read
  `research/claims.md` for the message you intend.
- Writes one report: `research/reviews/test-readers-<draft>-<date>.md` when `research/` exists,
  otherwise next to the draft. It never edits the draft unless you ask.

## Example prompts

- "Get fresh-reader feedback on `proposal.md` before I send it to the panel."
- "Test-read the introduction in `research/paper/introduction.md`. Will a non-specialist reviewer get
  the point?"
- "Will this email to the department head land? The ask is a second server by March."
- "Run the test readers again on the revised draft."
