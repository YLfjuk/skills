---
name: documentation
description: Standards for writing, reviewing, and organizing project documentation — external/user-facing docs, code comments and TSDoc/JSDoc, and public API documentation. Use this skill whenever creating, modifying, reviewing, or organizing any documentation, including writing a README or docs page, adding or editing code comments and doc comments, documenting a public API, adding `@since` tags, deciding where information belongs, or reviewing a diff that touches docs or comments. Also use it when the request only implies documentation — "explain what this module does in the code", "write up how to use this CLI", "add comments here", "clean up the docs" — even if the words "documentation" or "docs" are never used.
---

# Documentation

Use this skill whenever creating, modifying, reviewing, or organizing project documentation.

Documentation should be **purposeful, concise, accurate, and appropriate to its audience**.

The goal is not to document everything that exists. The goal is to communicate information that is useful at the place where it is presented.

## Core principles

### Document value, not existence

Do not document something merely because it exists in the codebase.

Before adding documentation, ask:

> What useful information does this provide that the reader cannot easily get from the thing itself?

Prefer documentation that communicates:

* Intent
* Behavior
* Constraints
* Usage
* Important decisions
* Non-obvious edge cases
* Guarantees and assumptions

Avoid documentation that merely restates:

* Names
* Types
* Obvious behavior
* Straightforward implementation
* Information already clearly expressed elsewhere

### Don't over-explain

Documentation should contain the **minimum useful explanation**.

Avoid:

* Long introductions
* Repetitive explanations
* Filler
* Excessive examples
* Exhaustive information dumps
* Explaining concepts that are already obvious
* Explaining implementation details that don't affect usage

Prefer high information density over completeness for its own sake.

### Document intent over mechanics

When explaining code or behavior, prefer explaining **why** something is done and **what contract it provides** rather than describing how the implementation works.

Implementation details should only be documented when they are relevant to understanding, using, or safely modifying the system.

### Prefer clear code over comments

When documentation is only necessary because code is difficult to understand, first consider whether the code itself should be improved.

Prefer:

* Better naming
* Simpler control flow
* Explicit types
* Better structure

over comments that explain unnecessarily complicated code.

### Maintain a source of truth

Avoid duplicating the same information across multiple locations.

When information must appear in multiple representations, establish a canonical source and derive or reference the other representation where possible.

In particular, distinguish between:

* Source-level API documentation
* Generated API reference
* Conceptual/user-facing documentation
* Project requirements and specifications

These serve different purposes and should not simply duplicate one another.

### Respect audience boundaries

Classify documentation by who can encounter it, not only by where it is written.

A source comment is external documentation when consumers can see or interact with it through an IDE hover, generated API reference, declaration output, package metadata, or similar tooling. Keep such comments self-contained and consumer-facing. They must not refer to repository notes, internal documentation, private discussions, or other maintainer-only context.

Internal comments may reference repository-local notes or internal documentation when consumer-facing tooling cannot expose the reference. Include enough nearby context for the code to remain understandable if the reference moves or becomes unavailable.

Repository notes should primarily record facts, decisions, requirements, and rationale owned by the repository. Do not make issues or discussions in external projects part of their explanation. When external behavior matters, record the relevant behavior and its local consequence directly in the note.

### Keep rationale project-owned

Explain the project on its own terms, using its requirements, constraints, behavior, and intended outcomes.

Do not turn private conversations, personal preferences, unpublished concerns, or incidental author context into project rationale. Include such context only when the user explicitly requests it and it is appropriate for the document's audience.

Prefer direct explanations of local value and consequences. Do not justify a decision by criticizing or dismissing another tool, project, or approach.

Name an alternative only when the comparison helps the reader make a current decision or prevents a plausible harmful change. State the concrete tradeoff or local consequence rather than attributing the choice to personal preference.

### Don't restate what you don't own

Before writing documentation, ask:

> If this were wrong, where would someone go to correct it?

If the answer is anywhere other than the thing being documented, the text is a copy. Copies go stale silently, because the owner changes without knowing the copy exists.

Common owners that are not this code:

* The language or runtime
* A tool or library — its docs, error messages, config schema, rule descriptions
* A project standard, convention guide, or loaded skill
* A project note: requirements, decision records, design docs
* The version history — what changed, in what order, what it used to be

Ownership elsewhere does not forbid the documentation. It demotes it: the text must now earn its place on a **local consequence** — something about *this* code that goes wrong if a reader acts on the general knowledge alone.

Where a local consequence exists, state the consequence and cut the explanation. The owner's documentation is one search away; this copy is not maintained.

```
✗  Naming any plugin replaces the tool's default set, and `react` is one of the
   defaults, which is why rules like exhaustive-deps and no-array-index-key stop
   running when plugins are listed explicitly.
✓  Listing any plugin replaces the defaults — `react` is one, so it must be named.
```

This is the difference between a comment that survives and one that merely repeats: the first tells a reader what will break here, the second re-teaches a subject that has an owner.

**This test is not an argument for fewer comments.** It is a filter on which ones. Code that looks wrong, redundant, or removable because it works around external behavior has a local consequence by definition — someone will try to delete it — and must carry a comment saying what breaks if they do. Applying this principle as a mandate to strip comments produces a worse codebase than the restating it was meant to prevent.

```
✓  The upstream rule exempts `ref.current`, but this linter only recognises a ref
   from `useRef`; ours comes from `useTerminal`. Inline disables don't suppress it.
✓  A leading `@` splices the entry into the parent archive instead of adding it —
   the bundle silently comes out incomplete.
```

Both name an owner elsewhere. Both stay, because both stop a reader from breaking something.

### Follow existing conventions

Before creating documentation:

* Inspect existing documentation
* Follow established terminology
* Follow established formatting
* Reuse existing documentation patterns
* Follow established tag usage
* Use existing documentation tooling where available

Do not introduce a new documentation style or convention when the project already has one.

### Keep documentation accurate

Documentation is part of the project and must remain consistent with the implementation.

When behavior changes:

* Update affected documentation
* Remove obsolete documentation
* Remove stale comments
* Do not preserve historical explanations that no longer apply

A misleading explanation is worse than no explanation.

---

# Documentation types

This skill covers three closely related forms of documentation:

* **External documentation** — documentation intended for users of the project.
* **Code documentation** — documentation embedded in or immediately alongside the source code.
* **API documentation** — documentation of the public programmatic interface, including both source-level documentation and its generated reference.

Use the appropriate reference for detailed guidance:

* `references/external.md`
* `references/code.md`
* `references/api.md`

These references build on the principles defined in this skill.

## External documentation

External documentation should help users understand and successfully use the product.

Prioritize:

1. Understanding what the product does
2. Getting started
3. Common workflows
4. Important concepts
5. Useful reference information
6. Advanced functionality

Do not expose implementation details simply because they are technically interesting.

For detailed guidance on user-facing documentation, see `references/external.md`.

## Code documentation

Code documentation includes:

* TSDoc / JSDoc
* Inline comments
* API documentation comments
* Comments explaining non-obvious behavior

Code documentation should remain close to the code it describes and should not become a substitute for project requirements or specifications.

For detailed guidance, see `references/code.md`.

## API documentation

Public programmatic APIs should be documented accurately and consistently.

API documentation has two related but distinct purposes:

### Source-level documentation

Documentation attached to the public API in the source code.

This should provide meaningful information to developers and serve as the source for generated documentation where applicable.

### Generated API reference

The generated API reference should expose the public API in a comprehensive, navigable form.

It may be considerably more exhaustive than the conceptual documentation.

Do not duplicate the generated API reference throughout the user-facing documentation.

For detailed guidance, see `references/api.md`.

---

# Requirements and specifications

Project-level requirements should have a canonical location.

Functional requirements, non-functional requirements, product requirements, architectural requirements, and similar project-level information should **not be scattered throughout the implementation as comments**.

If the project uses a `<root>/notes` directory, project requirements and related information may live there.

Keep notes primarily local to the repository. They should explain repository-owned requirements, decisions, and consequences without sending readers to an external project's issue tracker or discussion for essential context.

Code comments may reference a requirement when useful, but the requirement itself should have a canonical location.

Only internal comments may point to repository notes or internal documentation. Comments exposed to consumers through IDEs, generated reference, declaration output, or other tooling must state the public contract directly and may link only to public documentation intended for those consumers.

---

# Documentation hierarchy

Prefer the following relationship:

**Project requirements / notes**

→ Define what the project must do.

**Source code**

→ Implements the requirements.

**Code documentation**

→ Explains non-obvious intent, contracts, and behavior.

**Generated API reference**

→ Exposes the public programmatic API.

**External documentation**

→ Teaches users how and when to use the product.

These layers should complement one another rather than duplicate one another.

---

# Review checklist

Before considering documentation complete, verify:

* Does this information provide meaningful value?
* Is it appropriate for the intended audience?
* Can consumers encounter this through an IDE, generated documentation, declarations, or package tooling? If so, is it self-contained and free of internal references?
* Does the rationale come from project requirements and consequences rather than private or personal context?
* If another tool or approach is named, does the comparison explain a concrete tradeoff the reader needs?
* Could it be shorter without losing useful information?
* Does it explain something that isn't already obvious?
* Am I documenting intent or behavior rather than merely restating implementation?
* Am I duplicating information unnecessarily?
* Does this explain something whose canonical owner is elsewhere — a tool, a standard, a note, the history? If so, is there a local consequence that earns it?
* Conversely: does this code work around external behavior, and would a reader who doesn't know that behavior break it? Then it needs a comment.
* Is there a more appropriate canonical location?
* Does it follow existing project conventions?
* Is it accurate according to the current implementation?
* Will it be easy to keep accurate as the project changes?
* If this concerns a public API, is the public contract properly documented?
* If this is project-level requirements information, does it belong in the project's notes/specification area instead?
* If this is a repository note, does it preserve the relevant local context without relying on an external project's issue or discussion?

When in doubt, prefer **less documentation with higher information density**.
