---
name: documentation
description: Write, review, and organize project documentation, including READMEs, guides, release notes, code comments, TSDoc/JSDoc, and public API reference. Use when editing documentation or comments, reviewing a diff that touches them, or deciding what to document and where it belongs.
---

# Documentation

Write the explanation the reader needs at the place they need it. Document useful intent, behavior, constraints, and usage. Names, types, and obvious implementation usually need no prose.

## Choose the reference

Read only the references relevant to the task. Their scopes overlap when public API comments feed a generated reference or a user guide demonstrates an API.

* [Code documentation](references/code.md): comments on public or internal symbols, implementation comments, configuration, and module headers.
* [API documentation](references/api.md): supported public interfaces, generated reference, and required `@since` tags. For source comments, also read the code reference.
* [External documentation](references/external.md): READMEs, guides, release notes, CLI documentation, examples, and interactive documentation.

## Shared rules

### Add information the reader cannot easily infer

Use the minimum explanation that preserves useful meaning. Describe consumer contracts for symbols, and reasons or failure consequences for maintainers. A symbol's consumers include code that uses it within the repository. Explain implementation mechanics only when they matter to usage or safe changes.

If confusing code is the only reason a comment is needed, first consider clearer names, types, control flow, or structure. Keep warnings about non-obvious behavior even when the code is otherwise clear.

### Put information in its appropriate home

Give each kind of information an appropriate home:

* Requirements and decision records define what the project must do and why.
* Source comments explain symbol contracts and local implementation constraints.
* Generated API reference exposes supported interfaces and precise signatures.
* User guides teach tasks, concepts, and workflows.
* Release notes explain changes and any action existing users need to take.

Derive or link to canonical information where practical. These documents complement each other; a requirement does not replace a necessary API contract or local warning.

Project requirements belong in the project's existing notes or specification area, such as `<root>/notes` when present. Do not scatter them through source comments or create a new notes structure merely to follow this skill.

Explain decisions through project requirements, constraints, and consequences. Include private discussions or personal preferences only when the user requests that context and it suits the audience. Mention a rejected alternative only when its concrete tradeoff helps a current decision or prevents a plausible harmful change.

### Respect audience boundaries

Decide who can encounter the text from the actual output. A comment may reach consumers through IDE hovers, generated reference, declaration files, or package tooling, even if its implementation looks internal.

When one source feeds several outputs, check the complete text each audience receives. A heading or metadata label does not necessarily filter the content for that destination.

Consumer-visible comments must stand on their own, avoid maintainer-only context, and link only to public documentation intended for those readers.

Doc comments must not reference repository notes or internal documentation, whether the symbol is public or internal. Regular implementation comments may do so only when the reference cannot reach consumers. Include the essential local constraint beside the code so it remains understandable without the link.

### Explain local consequences of external facts

Ask who owns a fact and where someone would correct it. Language rules, tool behavior, project standards, and version history usually have a source elsewhere. Repeat only the detail needed to explain a local consequence; avoid copying their manuals or narrating past changes.

Keep the essential explanation in the documentation itself. An external issue or discussion can support it, but should not be needed to understand the project's decision or constraint.

A workaround or redundant-looking guard needs a comment when a reader could otherwise remove it and break the code. State what breaks and the external behavior responsible. This rule filters redundant explanations; it does not justify stripping necessary warnings.

### Follow conventions and keep facts current

Inspect nearby documentation for terminology, formatting, tag usage, and tooling. Reuse established patterns rather than introducing a competing style.

Check documentation against the current implementation. Update affected explanations when behavior changes and remove obsolete comments or historical rationale that no longer applies.

## Review

Before finishing, check that:

* The text adds useful information for its audience and sits where readers need it.
* Contracts and local warnings remain clear without duplicating specifications or reference manuals.
* Doc comments and consumer-visible text contain no internal references.
* Rationale states project requirements and consequences rather than incidental author context.
* Links, examples, signatures, and claims match current behavior.
* The edit follows project conventions and contains no unnecessary explanation.
