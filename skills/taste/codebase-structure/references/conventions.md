# Filesystem Conventions

These are filesystem conventions, **not architectural layers**. A `utils/` or `configs/` directory doesn't represent a conceptual responsibility.

- [Cohesive files](#cohesive-files)
- [Naming](#naming)
- [Kind-named folders](#kind-named-folders)
- [Configs](#configs)
- [Constants](#constants)
- [Utilities](#utilities)
- [Helpers](#helpers)
- [Type re-exports](#type-re-exports)
- [Tests](#tests)
- [Generated and tool-owned files](#generated-and-tool-owned-files)

---

## Cohesive files

A file should represent one cohesive concept. Multiple exports are fine when they form a coherent unit:

```text
colors/guards.ts
```

```ts
isWarmColor();
isColdColor();
```

Unrelated utilities get separate files:

```text
utils/
  is-defined.ts
  is-nullish.ts
```

**A single caller is not a reason to keep a distinct concept inline.** Usage count helps decide whether code stays with its owner or moves to a shared layer; it does not decide whether the concept deserves its own file. Extract a component, type, schema, or function when a focused file makes its responsibility easier to name, find, or understand.

**Do not split files merely to enforce one export per file.** That permits cohesive sets to stay together; it does not justify combining unrelated concepts. When a file outgrows a single cohesive concept, promote it to a directory:

```text
constants/{api.ts,pagination.ts}
utils/{format-date.ts,parse-query.ts}
```

---

## Naming

Default convention is `kebab-case`, components included:

```text
some-other-name.ts
some-component.tsx
```

File names shouldn't repeat their containing folder — the folder already provides context:

```text
feature/component.tsx           ✓
feature/feature-component.tsx   ✗
enums/status.ts                 ✓
enums/status.enum.ts            ✗
```

**Exception: keep a kind suffix when an external convention already establishes it.** Tooling (`vite.config.ts`, `eslint.config.ts`), test runners (`users.test.ts`), or a framework (`users.controller.ts`, `users.module.ts` in NestJS). Parity with files you don't control is worth more than the no-repeat rule:

```text
configs/env.config.ts           ✓  matches vite.config.ts, eslint.config.ts
controllers/users.controller.ts ✓  NestJS convention
```

**Framework conventions always win** over anything in this guide — including naming, not just directory structure.

---

## Configs

```text
apps/{app}/src/configs/
  env.config.ts
  db.config.ts
  app.config.ts
```

`env.config.ts` reads and validates environment variables:

```ts
const ENV = EnvSchema.parse(process.env);
```

Other config files derive subsystem configuration from the validated environment, or define constant configuration:

```ts
// db.config.ts
export const dbConfig = {
	username: ENV.DB_USERNAME,
	password: ENV.DB_PASSWORD,
} as const;
```

```ts
// app.config.ts
export const appConfig = {
	name: "my-app",
	title: "app",
} as const;
```

Config files define configuration, not unrelated application logic.

Use `satisfies` when a config object must conform to a definite shape without losing its inferred type.

---

## Kind-named folders

`constants/`, `utils/`, `types/`, `enums/`, `schemas/`, `mappers/`, `helpers/`, `components/`, `services/` all follow one shape:

```text
{kind}/{name}.ts        enums/status.ts, schemas/user.ts, mappers/color-by-status.ts
```

Plural folder, singular file, **no kind suffix** — the folder is the context. The exception is a suffix an external convention already establishes (see [Naming](#naming)).

These are technical subdirectories, **not layers**. They appear at whatever layer owns the code — `core/utils/`, `foundation/breadcrumbs/types/`, `features/users/enums/`. Placement follows the narrowest-owner rule from SKILL.md; the folder name says what kind of thing it is, not where it belongs.

**One file or a folder?** The cohesive-file rule decides it, and the answer differs by kind for a reason:

- Things that **compose** can share a file. Field schemas built up into a larger schema are one concept, so `schemas.ts` is valid and promotes to `schemas/` when distinct schemas become independently meaningful. Same for `constants.ts` and `utils.ts`.
- Things that **don't compose** get one per file. Two unrelated enums in `enums.ts` are just two unrelated things sharing a file, so enums go straight to `enums/{name}.ts` from the first one.

Subset enums spread into a parent are the composing case, so they belong in one file together.

## Constants

Small cohesive collections use `constants.ts`. Promote to a directory when distinct groups become independently meaningful:

```text
constants/
  api.ts
  pagination.ts
```

Avoid a separate file per constant unless the constant is an independently meaningful module.

---

## Utilities

```text
utils/
  format-date.ts
  parse-query.ts
  is-defined.ts
```

A single `utils.ts` is fine for a small cohesive set.

Utilities should be generic **relative to their containing layer** — a util inside `features/users` may know about users; a util in `core` may not. Feature- or domain-specific business logic belongs to its owner, not to a `utils` directory.

---

## Helpers

Helpers carry application- or business-oriented semantics but don't warrant a larger module. They may live as `helpers.ts`, a `helpers/` directory, or colocated with their owner:

```text
features/{feature}/helpers/
```

Don't let `helpers` become a dumping ground. If helper logic has a clear feature or domain owner, put it there.

---

## Type re-exports

Type-only re-exports must be explicit:

```ts
export type * from "./types";
```

Never use a plain `export *` for a type-only re-export.

---

## Tests

Tests follow the ownership of the code they test. Tests for reusable package functionality live with that package. **Do not move tests into a more generic layer just because the code under test is shared.**

**Two layouts are equally acceptable.** Colocated beside the code:

```text
features/users/users.test.ts
```

or a `test/` sibling of `src/` at the package root:

```text
packages/core/
  src/
    bundle.ts
  test/
    bundle.test.ts
```

Neither is the default. Pick one per package and stay consistent within it — mixing both in one package is the thing to avoid.

**Test helpers live with the tests, not in `src/`.** A fake, fixture, or builder used only by tests is test infrastructure; putting it in `src/` ships it to consumers.

```text
packages/core/test/fake-registry.ts    ✓
packages/core/src/fake-registry.ts     ✗
```

Repository-wide integration or e2e tests may live in a dedicated test package or app when they exercise multiple architectural responsibilities.

---

## Generated and tool-owned files

Files whose structure is dictated by a framework, build tool, or generator follow that tool's organization instead of these conventions:

```text
vite.config.ts
next.config.ts
eslint.config.ts
```

Framework-specific directories likewise use whatever structure the tool requires.

**Do not restructure tool-owned files to conform to this guide** when doing so conflicts with the tool's conventions.
