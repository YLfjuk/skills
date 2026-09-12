# API Documentation

Use this reference when documenting a public programmatic API.

API documentation consists of two related layers:

1. **Source-level API documentation** — documentation attached to public APIs in the source.
2. **Generated API reference** — the externally navigable reference generated from the public API.

These layers should complement each other.

## Public API

Only intentionally supported public APIs should be presented as part of the public API reference.

Do not expose internal implementation details merely because they are technically exported.

When determining the public API, consider:

* Public package exports
* Supported entry points
* Public types
* Documented extension points
* Explicitly supported interfaces

Follow the project's existing conventions for determining public API boundaries.

## Source-level documentation

Source-level documentation should describe the public contract.

Where useful, document:

* Purpose
* Behavior
* Parameters
* Parameter constraints
* Return values
* Errors
* Side effects
* Important assumptions
* Non-obvious semantics
* Usage requirements

Do not reproduce implementation details unless they affect the public contract.

Source-level documentation that can reach consumers through IDE hovers, generated reference, declaration output, package metadata, or similar tooling is external documentation. It must stand on its own and must not refer to repository notes, internal documentation, private discussions, or maintainer-only context. Link only to public documentation intended for the same consumers.

## `@since`

Every newly introduced public API must include a `@since` tag with the appropriate future version.

If the version is not yet known:

```ts
/** @since @next */
```

Replace `@next` with the actual release version once known.

An automated process may perform this replacement.

## Generated API reference

Prefer generating the API reference from the source and its type information rather than manually maintaining a duplicate API specification.

Use the project's existing API documentation tooling when available.

The generated reference should accurately expose the supported public API, including relevant:

* Functions
* Classes
* Types
* Interfaces
* Enums
* Constants
* Configuration objects
* Parameters
* Return types
* Generic parameters
* Relevant overloads

Do not expose private or internal implementation details.

## API reference vs conceptual documentation

The API reference may be comprehensive.

Conceptual documentation should not be.

For example, a conceptual page might say:

> Use `createBundle` when integrating bundling into another application.

and provide a concise example.

The API reference can then provide:

* The complete signature
* All parameters
* Types
* Return type
* Overloads
* Detailed API-level documentation

Do not duplicate the complete generated reference in the conceptual page.

## Examples

Include examples where they meaningfully help developers understand important APIs.

Prefer realistic examples over examples designed solely to demonstrate every possible type or overload.

Simple APIs may need little or no additional prose beyond their source documentation.

## API consistency

Follow the project's established conventions for:

* Naming
* Documentation tags
* Version annotations
* Export boundaries
* Generated documentation
* Examples
* Linking between guides and reference material

Do not invent a new API documentation convention when an existing one is available.

## Review checklist

Before completing API documentation:

* Is every intended public API represented?
* Are internal implementation details excluded?
* Are public APIs appropriately documented at the source?
* Is all consumer-visible source documentation self-contained and free of internal references?
* Does each new public API have the required `@since` information?
* Is generated documentation used where appropriate?
* Are conceptual docs avoiding unnecessary duplication of the API reference?
* Are signatures and types accurate?
* Are examples useful rather than exhaustive?
* Does the documentation follow established project conventions?
