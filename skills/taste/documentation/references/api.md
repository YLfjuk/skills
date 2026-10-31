# API documentation

Use this reference for supported public programmatic interfaces and their generated reference. For doc comments on public or internal symbols, also read [code documentation](code.md).

For usage guides and examples, read [external documentation](external.md).

## Define the supported API

Only intentionally supported interfaces belong in the public API reference. A technical export alone does not establish support.

Follow the project's boundaries, including public package exports, supported entry points, public types, and documented extension points. Exclude internal implementation details.

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

## Review

Apply the shared review in [SKILL.md](../SKILL.md), then check:

* Every supported public API is represented, and internal details are excluded.
* Source explanations and generated signatures match the implementation.
* Each new public API has the required `@since` information.
* Generated reference uses established tooling and conventions.
