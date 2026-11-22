---
name: orwell-clear-writing
description: Draft or revise English prose using George Orwell's six rules for clear, honest writing. Use when writing or editing documentation, READMEs, PR or review summaries, reports, emails, articles, and other prose that benefits from plain language, tighter sentences, active voice, and less jargon or cliché.
---

# Orwell Clear Writing

Make the meaning easy to see without making the writing flat, evasive, or less accurate.

## Workflow

1. Identify the audience, purpose, and non-negotiable meaning. Preserve necessary technical or legal precision.
2. When drafting, write plainly from the start, then run one pass against the six rules and editing checks below. When revising, apply them directly and prefer the smallest change that improves clarity.
3. Read the result as a whole. Restore a rule-breaking construction when it is the clearest, most natural, or least misleading choice.
4. Return the text according to `## Output modes` below. Flag unresolved ambiguities only when meaning could change; do not explain minor substitutions.

## The six rules

Use the first five as concrete tests and the sixth as an override:

1. Never use a metaphor, simile, or other figure of speech that you are used to seeing in print. Replace stale imagery with a concrete fact or a fresh, necessary image.
2. Never use a long word where a short one will do.
3. If it is possible to cut a word out, always cut it out.
4. Never use the passive where you can use the active.
5. Never use a foreign phrase, a scientific word, or jargon if you can think of an everyday English equivalent. Keep terms of art when replacing them would reduce accuracy, and define them briefly for a general audience unless adding definitions would alter legal or contractual meaning.
6. Break any of these rules sooner than say anything outright barbarous. Do not sacrifice precision, tone, rhythm, safety, or fairness merely to comply mechanically.

## Editing checks

Use these tests while revising:

- Replace abstractions with the person, action, object, amount, date, or decision they conceal when known; otherwise flag what is missing.
- When drafting an explanation of an abstract or technical point, add a concrete example (label any illustrative figures as examples).
- Cut empty openings, duplicated qualifiers, filler transitions, and double negatives (`not un-`) when plain wording keeps the same meaning and degree.
- Prefer a verb over a noun built from one: write "decided" rather than "made a decision."
- Change passive voice only when the actor is known and matters to meaning or accountability. Retain it when the actor is unknown, irrelevant, deliberately protected, or the receiver of the action is the focus.
- Do not conceal responsibility: prefer "We delayed the release" to "The release was delayed" when the author is responsible and naming them is relevant.
- Preserve quoted text, defined terms, code identifiers, names, numbers, and requirements unless asked to change them.
- Do not treat simplification as a license to change the writer's claim: flag a vague claim or a missing actor (when the actor matters to meaning or accountability) instead of inventing one, and ask a concise question or mark the uncertainty when intended meaning cannot be recovered.

## Output modes

Match the request:

- **Rewrite (default when revising):** Return only the polished text unless a rationale is requested — except a concise one-line flag or question when missing facts or unclear meaning would otherwise force a guess.
- **Edit with explanation:** Return the revision, then up to 6 key changes grouped by rule (or note when no change is needed).
- **Draft:** Write directly in plain English, then run one editing pass using the checks above.
- **Light edit:** Keep the author's voice and structure; fix only clear wordiness, clutter, or ambiguity without rewriting sentences that already read clearly.

## Examples

- "At this point in time, the implementation of the policy is being undertaken by the team." → "The team is now carrying out the policy."
- "The results were not unexpected." → "The results were expected."
- "We will leverage a synergistic framework." → *Flag: vague claim — ask what specific action or tool will be used instead of inventing one.*
