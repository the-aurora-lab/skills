---
name: test-readers
description: Find out how real readers will receive a draft before you send it. Runs the draft past blind, fresh-context reader personas (Newcomer, Expert, Editor, Decision-maker, Skimmer), checks whether the intended message landed, and returns a tactful, severity-ranked report with a concrete fix for every issue. Use when asked for reader feedback, a test read or fresh eyes on a draft, whether a document will land with its audience, or before sending a paper, proposal, report or email that matters.
---

# Test readers

Authors cannot test their own drafts: they know what each sentence was meant to say, so they read that
instead of what is on the page. A reader with a fresh context has none of that knowledge, so its
reading is a genuine first reading. This skill builds such readers, gives each one only the draft, and
turns what they report into a ranked list of fixes.

## Reading mode: fresh readers or a careful fallback

- **If your agent can run subagents or start a fresh session** (one that sees only what you hand it),
  give each persona its own, with nothing but the blind packet from step 3. This is the intended mode:
  each reader genuinely meets the draft for the first time.
- **Otherwise**, read as each persona yourself, one at a time, in separate passes. You already know the
  author's intent, so you will be too kind: in each pass, quote the draft before reacting, report only
  what the page says, and assume the reader has none of the context from this conversation. State in
  the report's opening line that the readings were simulated in one context, and recommend one real
  reader from the audience before anything high-stakes is sent.

## Steps

1. **Scope the review.** Settle these with the user, or infer them from the context:
   - What the draft is and who its real readers are.
   - The kind of feedback wanted: clarity, accuracy, language, or all three.
   - The depth: overall structure or line by line. An early draft gets structure feedback; a
     near-final draft gets line-level feedback too.
   - The intended main message and the action the reader should take. If the user has not said and
     can be asked, ask: they are the yardstick for step 5. If research/claims.md exists and the draft
     is part of that paper, use it to fill in the intended message.

   When the user cannot be asked, infer what is missing and list the assumptions at the top of the
   report. Done when all four are written down.

2. **Pick personas** from *Personas*. Always include the Newcomer when anyone outside the author's
   circle will read it. Add the Expert for technical or factual claims, the Editor for language, the
   Decision-maker when the draft asks for money, approval or action, and the Skimmer for anything
   longer than two pages or anything read in a hurry. Three to five personas is typical.

3. **Prepare a blind packet** for each persona: the path to the draft (or the rendered slides or PDF)
   plus the persona prompt. Leave out the author's intent, the outline, earlier versions, and
   explanations such as "section 3 is meant to...", because they bias the test. Only the Expert may
   get the domain background needed to judge accuracy (the field, not what the draft is trying to say).

4. **Run the readings**, in parallel when your agent supports it, using *Persona prompt*. Done when
   every persona has reported.

5. **Comprehension probe.** Put each persona's "main message" and "what I'm expected to do" next to
   the intended ones. A mismatch is a top finding even when no persona complained: the draft says
   something other than what was intended.

6. **Merge and rank.** Deduplicate the findings and rank them by severity:
   - *blocks understanding*: a reader could not follow or could not act.
   - *misleads*: a reader came away believing something the author did not mean, or that is not true.
   - *slows the reader*: a reader got there, but had to reread, look something up or guess.
   - *polish*: wording, spelling and consistency that do not change the meaning.

   Sort the issues table strictly by these bands, most severe first, and within a band by how many
   readers tripped on the passage and how close it sits to the main message or the ask. Two or more
   personas tripping on the same passage can move it up one band. A comprehension mismatch from step
   5 goes in the table too, as *misleads* or worse. Check every number, total and count the personas
   questioned against the draft itself before reporting it. Write the report
   (*Report format*). Done when every issue states where it is, its object (content, structure or
   language), its nature (must fix or suggestion), what the reader experienced, and a concrete fix or
   question.

7. **Save and offer next steps.** Save the report (see *Files*) and give the user the overall verdict
   and the top three issues in the chat. Offer to apply the fixes or to leave the draft to the author;
   change the draft only when asked. After major changes, run a second round with new readers: the
   earlier ones have now seen the draft.

## Personas

| Persona | Stands for | Reads by | Looks for |
|---|---|---|---|
| **Newcomer** | The least specialised intended reader, reading months from now with no context | Reading everything once, in order | Where they got lost, words they would have to look up, a missing *why*, the one sentence they remember |
| **Expert** | The most specialised reader | Reading everything critically | Wrong or unsupported claims, missing caveats, gaps in method, overstated conclusions, numbers that do not add up |
| **Editor** | A careful, fluent reader of the draft's language | Reading line by line | Grammar, spelling, word choice, terms used inconsistently, sentences that must be read twice |
| **Decision-maker** | The person who has to act | Reading the first page, then deciding | Is the ask clear? Cost, deadline, risk? Could they decide without turning the page? |
| **Skimmer** | Most real readers of long documents | Headings, first sentences, figures and captions only | Can the story be rebuilt from the skeleton? Which heading or caption misled them? |

You may adapt a persona to the real audience (for example, the Decision-maker as "a finance officer on
a funding panel"); keep its reading mode and checks.

## Persona prompt

```
You are a test reader: <persona description, from the table, adapted to the real audience>.
You have never seen this <document type> and know nothing about its subject beyond what a
<persona> would know. Read <path> the way this persona would: <reading mode from the table>.

Report, in this order:
1. The main message, in one sentence, as you understood it.
2. What you think you are expected to do or decide after reading.
3. Every place you got lost, slowed down, or stopped trusting the text. Quote the passage
   and describe what happened to you ("I didn't know what X meant", "I expected Y here",
   "I'd stop reading at this point").
4. <persona-specific checks from the "Looks for" column>
5. What worked well, and why.

Report your reactions as a reader; leave the rewriting to the author. Write in the first
person ("I found...", "I expected..."). Flag every term you would have to look up, and say
plainly where you would stop reading. Do not be polite on the author's behalf: a reader
who is confused or unconvinced says so.
```

Simulated readers know more and are kinder than real ones; the last paragraph of the prompt
counteracts that. Treat the result as a fast first test, and for high-stakes pieces recommend one real
reader from the audience as well.

## Report format

```markdown
## Test read: <draft name>

<One line: personas used, reading mode (fresh readers or simulated in one context), date.>
<Assumptions, if the scope was inferred.>

**Overall.** <2-3 sentences: does it reach its purpose with its audience? The biggest issue first.>

**Keep.** <what works and why: specific passages, structure, examples>

**Did the message land?**
| | Intended | Newcomer | Expert | Decision-maker |
|---|---|---|---|---|
| Main message | ... | ... | ... | ... |
| Expected action | ... | ... | ... | ... |

**Issues (most severe first)**
| # | Severity | Where | Object | Nature | What the reader experienced | Fix or question |
|---|---|---|---|---|---|---|
| 1 | blocks understanding | para 1 | structure | must fix | Decision-maker found no amount or deadline on the first page | Could we open with the ask? |

**Line notes** (if requested), grouped by section.
```

Use one column per persona in the message table. `examples/sample-report.md` is a complete report from
a real run on `examples/sample-draft.md`; read it when you want to see the expected level of detail.

## Tone

- Write to help, not to judge. Phrase issues as observations or questions: "Is the difference between
  *users* and *customers* intentional?" rather than "Be consistent". Use *we* for suggestions: "Could
  we lead with the result?"
- Point out the problem before offering a rewrite. A sentence replaced with no reason teaches nothing
  and feels like someone else's text.
- Use precise labels: content or form, error or suggestion, required or optional. Avoid vague marks
  such as "unclear" or "rework this" without saying what made it unclear.
- Give the strengths as concretely as the weaknesses. Authors revise more willingly when they know
  what to keep.

## When the author responds to feedback

If the user wants to argue with a finding, treat every reader reaction as data: "I found this unclear"
is true for that reader by definition. Ask what exactly was unclear instead of defending the text. The
author decides what to change, but only after understanding each reaction.

## Files

Everything here is optional; the skill also works on a draft pasted into the chat.

- **Reads** the draft you point to. With no path, and when the project has a `research/` folder, it
  looks for the manuscript in `research/paper/` (one file per section) and asks which part to test.
  It may read `research/claims.md` for the intended message (step 1), never for the readers' packet.
- **Writes** one file: the report, as `research/reviews/test-readers-<draft>-<YYYY-MM-DD>.md` when a
  `research/` folder exists, otherwise `test-readers-<draft>-<YYYY-MM-DD>.md` next to the draft (or
  only in the chat, if the user prefers). It never edits the draft unless asked.
- `references/research-folder.md` describes the shared `research/` folder convention. Read it when the
  project has a `research/` folder or the user asks how this skill fits with other tools.
