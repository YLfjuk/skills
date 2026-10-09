# Code documentation

Use this reference for doc comments, regular implementation comments, configuration comments, and module documentation.

## Choose the comment form

Choose by what the comment describes, not whether the symbol is exported:

* A doc comment describes a symbol's purpose, behavior, or contract for code that uses it.
* A regular comment explains the reason or constraint behind a particular line or block.

Internal functions, types, and constants may have doc comments under the same rules as public symbols. Their consumers are the code elsewhere in the repository that uses them.

A symbol whose name and type already communicate its purpose and behavior needs no explanatory doc comment, exported or not. Required metadata tags, such as `@since` for a new public API, are a separate concern.

Do not add comments that repeat names, types, obvious parameters or return values, or straightforward implementation. For example, this adds nothing:

```ts
// Get the user by ID
const user = getUserById(id);
```

If code is unnecessarily hard to understand, consider improving it before explaining it with comments. This does not remove the need to document non-obvious constraints.

## Document symbols

When writing a doc comment, describe the symbol's consumer contract: the behavior, guarantees, and constraints that code using it can rely on. Include purpose, errors, side effects, usage requirements, or non-obvious semantics where the signature leaves them unclear. Explain implementation details only when they affect that contract. Do not mechanically fill out every possible field.

A non-exported helper can still need a behavioral guarantee:

```ts
/** Preserves the first occurrence of each ID and its input order. */
function uniqueIds(ids: readonly string[]): string[] {
  return [...new Set(ids)];
}
```

Keep implementation rationale beside the implementation. Exporting a symbol does not make its internal details part of the supported contract.

Follow established doc-comment formatting and tag usage. For public API boundaries, generated reference, and the required `@since` tag, read [API documentation](api.md).

## Explain implementation constraints

Comment workarounds, ordering requirements, compatibility constraints, unusual values, or redundant-looking guards when a reader could otherwise change them and break behavior. State the local failure and enough of its cause to make the constraint understandable.

```ts
// Keep these requests sequential; the upstream service only guarantees
// ordering within a connection.
```

For external behavior, ask whether a competent reader who lacks that detail could break this code. If so, explain the consequence here. Otherwise, leave the language, runtime, or tool's explanation to its own documentation.

```ts
// Wait for the resize to settle; the observer's first tick reports
// the pre-layout box and would give us stale measurements.
```

Do not turn a comment into an external technology's reference manual. Likewise, do not record development history such as "deferred to a later pass" or "revised from the earlier approach" beside code. Explain the current constraint. Mention a rejected alternative only when someone might plausibly retry it and needs to know the concrete failure or tradeoff.

### Configuration

Leave self-describing configuration entries bare. Explaining a flag or rule's general meaning does not explain a local decision.

Comment locally surprising choices or entries whose modification would break something. For example, if a tool replaces its default plugin set when any plugin is listed:

```ts
// Listing plugins replaces the defaults; keep `react` listed so its rules still run.
```

## Keep internal references out of doc comments

Doc comments must not reference repository notes or internal documentation, even on internal symbols. This keeps the comment usable if the symbol later becomes public.

A useful note reference belongs in a regular comment beside the code it explains, and only when consumer-facing tooling cannot expose it. Consumer visibility depends on actual IDE hovers, generated reference, declarations, or package output, not just comment syntax or export status.

All consumer-visible comments must be self-contained and free of repository notes, private discussions, implementation plans, or maintainer-only terminology. Link only to public documentation for the same audience.

Keep the essential local intent or constraint in a regular comment even when it links to a note. Do not add requirement IDs merely to label implementing code, or replace a redundant explanation with a bare pointer. The canonical requirement does not replace a necessary symbol contract or local warning.

## Place comments where they help

Attach a doc comment directly to the public or internal symbol it describes. Put implementation rationale beside the line or block someone might change. Avoid distant comments that require searching to find their subject.

Default to no module header. A synopsis, ownership declaration, export inventory, or requirement pointer does not earn a header merely by being short.

A header is justified when a non-obvious constraint governs the whole module and cannot be attached more usefully to a symbol or block. One workaround belongs beside that workaround, even if it motivated the file's creation.

Required license notices and tooling directives are exempt from this default. Consumer-visible module documentation follows the same audience rules as other comments.

## Review

Apply the shared review in [SKILL.md](../SKILL.md), then check:

* Does the comment form match its subject rather than the symbol's visibility?
* Does the doc comment add behavior or constraints beyond the name and type?
* Does each workaround retain the warning needed to prevent a harmful change?
* Are comments close to their subjects and still accurate?
* Are note links confined to regular comments that cannot reach consumers?
* For public APIs, have the API reference's boundary and `@since` rules been applied?
