# API documentation

Use this reference for supported public programmatic interfaces and their generated reference. For doc comments on public or internal symbols, also read [code documentation](code.md).

## Define the supported API

Only intentionally supported interfaces belong in the public API reference. A technical export alone does not establish support.

Follow the project's boundaries, including public package exports, supported entry points, public types, and documented extension points. Exclude internal implementation details.

## Source-level documentation

For public APIs, describe the symbol's consumer contract. Add information the signature leaves unclear, such as constraints, errors, side effects, assumptions, and non-obvious semantics. The same content rule applies to internal doc comments.

Source comments should supply useful explanations for developers and feed generated reference where applicable. Self-describing symbols need no explanatory prose, but required metadata still applies.

Follow the code reference's comment-form, audience, and linking rules. Doc comments never reference repository notes or internal documentation. Any source documentation that reaches consumers must stand on its own and link only to public documentation for those readers.

## `@since`

Every newly introduced public API must include a `@since` tag for its release version. Follow the project's versioning and tag-format conventions.

If the future version is not yet known, use:

```ts
/** @since @next */
```

Replace `@next` with the actual release version once known, manually or through the project's automated process. Do not leave it after the version has been established.

## Generated reference

Prefer generating reference from source documentation and type information using the project's existing tooling. Avoid manually maintaining a duplicate API specification.

The reference should be comprehensive and navigable across supported functions, classes, types, interfaces, enums, constants, and configuration objects. Preserve accurate signatures, parameters, return types, generics, and relevant overloads. Exclude private and internal details.

## Guides and examples

A guide explains when and why to use an API and demonstrates a realistic task. Link to the reference for full signatures, options, and overloads instead of reproducing them.

Include examples when they clarify meaningful usage. Do not create examples solely to exercise every type or overload. Simple APIs may need little beyond their source documentation.

## Review

Apply the shared review in [SKILL.md](../SKILL.md), then check:

* Every supported public API is represented, and internal details are excluded.
* Source explanations and generated signatures match the implementation.
* Each new public API has the required `@since` information.
* Generated reference uses established tooling and conventions.
* Guides link to precise reference details without duplicating them.
