# Code Documentation

Use this reference when adding or modifying documentation within the source code.

This includes:

* TSDoc
* JSDoc
* Inline comments
* API documentation comments
* Comments explaining non-obvious behavior

## General rule

Do not add a comment merely because something can be commented on.

A comment should communicate information that cannot be understood as easily from the code itself.

## Don't restate the code

Avoid comments that merely repeat:

* Function names
* Variable names
* Types
* Obvious parameters
* Obvious return values
* Straightforward implementation

For example, avoid:

```ts
// Get the user by ID
const user = getUserById(id);
```

The code already communicates this.

## Explain intent

Prefer documenting:

* Why something is done
* Important constraints
* Non-obvious assumptions
* Design decisions
* Behavioral guarantees
* Important edge cases
* Compatibility requirements
* Reasons for unusual implementation choices

For example:

```ts
// Keep this request sequential because the upstream service applies
// a per-connection ordering guarantee.
```

The comment explains information that is not apparent from the code alone.

## Don't compensate for poor code

If a comment is needed primarily because the implementation is difficult to understand, consider improving the implementation first.

Prefer:

* Better naming
* Simpler logic
* More explicit types
* Better decomposition

over increasingly detailed comments.

## Third-party and external behavior

Comments may explain non-obvious behavior of:

* Third-party packages
* Frameworks
* Build tools
* Runtime environments
* Programming languages
* Platform APIs

This is appropriate when the external behavior explains an otherwise unusual implementation.

Keep these comments focused on the behavior relevant to this code.

Do not use comments as a general-purpose reference manual for the external technology.

### The discriminator

The external technology owns its own documentation. A comment restating it is an unmaintained copy. What the external documentation cannot contain is the effect on *this* code — that is what the comment is for.

**Write the comment when the code carries a plaster.** A workaround, an ordering constraint, a redundant-looking guard, a value that must be exactly this one, an override that appears deletable — anything shaped by external behavior that a reader cannot see from here. These comments are load-bearing: without them the next reader "cleans up" the plaster and reintroduces the problem. Say what breaks, briefly.

```ts
// Firing this before the resize settles gives us stale measurements —
// the observer reports the pre-layout box on first tick.
```

**Skip the comment when nothing here depends on knowing.** A setting that is unremarkable to anyone who knows the tool, a rule's own meaning, an idiom's definition. If deleting the comment leaves the code equally safe to change, delete it.

```ts
✗ // The automatic JSX runtime, so `React` does not need to be in scope.
  "react-in-jsx-scope": "off",
```

Applied in one line: **would a competent reader who doesn't know this external detail break the code?** Yes → comment it, from the consequence inward. No → the detail belongs to its owner.

### Two corollaries

**A setting's own meaning is never the comment.** Explaining what a rule, flag, or option does documents the tool, not the decision. Comment the ones whose choice is locally surprising; leave the rest bare.

**Project decisions are not code comments.** "Deferred to a later pass", "revised from the earlier approach", "chosen over X", and personal preferences record how the work went. They belong in the notes, the plan, or the changelog — not in the file, where they outlive the process they describe. Explain the code through its local requirements and consequences. Name a rejected alternative only when a future reader would plausibly retry it; state the concrete tradeoff or failure it would cause rather than criticizing the alternative.

### Configuration files

Configuration is where this fails most. Every line is a decision, which invites a justification per line — and most of those lines are self-describing to anyone who knows the tool.

Comment the entries that are locally surprising or that break something when changed. Leave the rest bare.

## Public APIs

Public APIs should have appropriate source-level documentation.

Document the public contract rather than the implementation.

Where relevant, document:

* Purpose
* Behavior
* Parameters
* Constraints
* Return values
* Errors
* Side effects
* Important usage requirements
* Non-obvious semantics

Do not document internal details simply because a symbol is exported.

## Consumer-visible comments

Treat a comment as consumer-facing whenever it can appear outside the repository through an IDE hover, generated API reference, declaration output, package metadata, or similar tooling. The syntax of the comment and the apparent visibility of the implementation are not reliable boundaries; use the output consumers can actually encounter.

Consumer-visible comments must be self-contained. Do not refer to repository notes, internal documentation, private discussions, implementation plans, or maintainer-only terminology. State the public contract directly, and link only to public documentation intended for the same audience.

Internal comments may reference repository-local notes or internal documentation when those references cannot be surfaced to consumers. Keep the essential local intent or constraint in the comment so that it remains useful without following the reference.

## `@since`

Every newly introduced publicly accessible API must include an appropriate `@since` tag.

Use the project's established versioning convention.

If the future release version is not yet known, temporarily use:

```ts
/** @since @next */
```

Replace `@next` with the actual version once it is known.

The project may instead use an automated process to perform this replacement.

Do not leave `@next` indefinitely once the release version has been established.

## Documentation tags

Doc comments must follow established project patterns and tag usage.

Before adding tags:

* Inspect existing documentation
* Follow existing ordering and formatting
* Reuse established terminology
* Use only tags that provide useful information

Do not introduce new conventions unnecessarily.

## Requirements and specifications

Do not use source-code comments as a distributed requirements document.

Functional requirements, non-functional requirements, product requirements, architectural requirements, and similar project-level information should have a canonical location.

If the project provides a `<root>/notes` directory, it may contain this information.

Notes should remain primarily about the repository itself. Do not use an external project's issue or discussion as part of the note's explanation. If an upstream behavior affects the repository, record that behavior and the local consequence directly.

An internal comment may explain how an implementation satisfies or works around a requirement and may point to its repository-local note when useful, but the requirement itself should not be duplicated throughout the codebase. Consumer-visible comments must not point to internal notes.

## Keep comments close to their subject

Place documentation as close as practical to the code it describes.

Prefer:

* A doc comment directly on a public API
* A short inline comment beside a non-obvious behavior
* A nearby explanation of an important constraint

Avoid distant comments that require readers to search the codebase to understand what they describe.

## Avoid duplication

Do not repeat information that is already clearly available through:

* Types
* Names
* Public API documentation
* Project notes
* Generated documentation

When duplication is necessary, maintain a clear source of truth.

## Keep comments maintainable

Comments should be updated when the behavior they describe changes.

Remove comments when:

* Their subject no longer exists
* The behavior has changed
* The explanation is now obvious from the code
* The original reason is no longer relevant

Avoid historical comments whose only purpose is to describe how the code used to work.

## Review checklist

Before adding a code comment, ask:

* Is this information non-obvious?
* Does it explain intent, behavior, or an important constraint?
* Could the code itself be made clearer instead?
* Is this really an implementation detail, or does it belong in project documentation?
* Can a consumer see this comment through an IDE, generated reference, declarations, or package tooling? If so, is it self-contained and free of internal references?
* Does the rationale come from local requirements and consequences rather than private or personal context?
* If another tool or approach is named, does the comparison prevent a plausible mistake or explain a tradeoff the reader needs?
* Am I duplicating information?
* Who owns this fact? If a tool, a standard, a loaded skill, a note, or the history owns it, does this code have a consequence that earns restating it?
* Does this code work around external behavior that a reader can't see from here? Then it needs a comment — say what breaks, not how the external thing works.
* Does it follow existing documentation conventions?
* Will it remain accurate?
* If this is a public API, does it have the required `@since` tag?

Prefer **short, high-value comments** over exhaustive explanations.
