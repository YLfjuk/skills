---
name: typescript-conventions
description: Standards for writing TypeScript code — types, enums, mappers, schemas, casts, iteration, compiler settings. Use when writing or reviewing any .ts or .tsx file, or configuring tsconfig, even if conventions aren't mentioned.
---

# TypeScript Conventions

Standards for how TypeScript is written here.

**Precedence.** Existing standards supersede this guide — highest priority first: mandatory organization policies, the repo's own standards, the framework's, and the language's. Refactor existing code toward these standards only when asked to.

## Type vs Interface

Prefer `type`. Reach for `interface` in two cases only:

- **Extending a large built-in type** — `React.HTMLAttributes`, `HTMLDivElement`, and similar.
- **A package's overridable interface**, where declaration merging is the point: consumers inject their own types into it.

```ts
// ✅ Default
type User = { id: string; name: string };

// ⚠️ Justified — extending a large built-in
interface Props extends React.HTMLAttributes<HTMLDivElement> {
  variant: "default" | "muted";
}
```

## Discriminated unions

Prefer a discriminated union over a bag of optional fields, where the shape genuinely has distinct cases:

```ts
// ❌ every consumer needs a non-null assertion
type Result = { isSuccess: boolean; data?: User; error?: Error };

// ✅ narrowing gives you the right fields
type Result = { isSuccess: true; data: User } | { isSuccess: false; error: Error };
```

This is what makes the no-cast rule livable — an optional-field bag forces `result.data!` after checking the flag, while the union hands you the field already narrowed. Once a discriminated union has more than two cases, use a string-literal discriminant (such as `status` or `kind`) rather than booleans.

Not every optional field wants this. A shape with independently optional fields that don't correlate is a bag of options, not a set of cases, and forcing a union on it invents distinctions that don't exist.

## Enums

**Never use the `enum` keyword.** It emits non-erasable runtime code (rejected by `erasableSyntaxOnly` in newer TypeScript versions), string enums are nominal (`"draft"` isn't assignable to `Status`), and `const enum` is fragile under `isolatedModules`.

```ts
enum DoNotUseThisKeyword {} // ❌
```

Use an object with `as const`, aliased by a type of the same name:

```ts
const Status = {
 Draft: "draft",
 Published: "published",
 Archived: "archived",
} as const;

type Status = ValueOf<typeof Status>;
```

The const and the type share a name deliberately — `Status.Draft` is the value, `Status` is the type, and there's one name to remember.

### ValueOf

```ts
type ValueOf<T> = T[keyof T];
```

Use the project's existing helper if there is one, or import from `type-fest` if it's already a dependency. Otherwise define it.

`(typeof Status)[keyof typeof Status]` is the same thing inline and is fine where a helper isn't available.

### Naming

- **Name** — `PascalCase`, **singular**. The type takes the same name.
- **Keys** — `PascalCase`. Acronyms are ordinary words: `StdLib`, `HttpStatus`, `XmlParser`. Never `SCREAMING_SNAKE_CASE`, never mixed within an enum.
- **Values** — `kebab-case` where context allows. External APIs and wire formats dictate their own.

```ts
const Source = {
 Local: "local",
 StdLib: "std-lib",
 HttpCache: "http-cache",
} as const;

type Source = ValueOf<typeof Source>;
```

### Object enums over array enums

Object form is singular and aliases cleanly. Derive the array when you need it:

```ts
const Statuses = Object.values(Status);
```

Name object-enums singular, array-enums plural. The type is always singular.

### Subset enums

When you need an enum covering part of another's values, pick by whether the subsets overlap.

**Split and spread** — when subsets are disjoint:

```ts
const SubValue = { A: "a", B: "b" } as const;
type SubValue = ValueOf<typeof SubValue>;

const OtherSubValue = { C: "c", D: "d" } as const;
type OtherSubValue = ValueOf<typeof OtherSubValue>;

const Value = { ...SubValue, ...OtherSubValue } as const;
type Value = ValueOf<typeof Value>;
```

These belong in one file — they're one concept expressed in parts.

**Derive an array subset** — when subsets overlap or the full enum is authoritative:

```ts
const Value = { A: "a", B: "b", C: "c", D: "d" } as const;
type Value = ValueOf<typeof Value>;

const SubValues = [Value.A, Value.B] as const;
type SubValue = (typeof SubValues)[number];
```

## Mappers

A consistent mapping between two domains — enum to label, status to color, DB value to UI value.

- **Name** — `camelCase`, `yByX`, read as **y = f(x)**: `colorByStatus` maps a status to a color.
- **Type** — `Record<X, Y>`. Keys are the input (`x`), values the output (`y`).
- **Never type-cast.** Use `as const satisfies`, which checks exhaustiveness while keeping literal inference.

```ts
const colorByStatus = {
 [Status.Draft]: "gray",
 [Status.Published]: "green",
 [Status.Archived]: "red",
} as const satisfies Record<Status, string>;
```

`satisfies` is what makes this work: annotating the variable would widen `'gray'` to `string`, and casting would skip the exhaustiveness check that catches a missing status when the enum grows.

## Schemas

A schema is a value and a type, like an object-enum — but unlike an enum, the clean name belongs to the type, so the schema carries a `Schema` suffix.

Which one drives the other depends on whether the project treats schemas or types as its source of truth. This is a project-level decision, not a per-schema one — check what the surrounding code does and match it.

**Schema drives type** — schemas are authoritative (using your schema library's inference helper, e.g., `z.infer` or `InferOutput`):

```ts
const ValueSchema = /* ... */;
type Value = Infer<typeof ValueSchema>;
```

**Type drives schema** — types are authoritative and the schema is one representation of them:

```ts
// in the types file
type Value = { /* ... */ };

const ValueSchema = /* ... */ satisfies Schema<Value>;
type ValueShape = Infer<typeof ValueSchema>;
```

`satisfies` catches most drift; inferring back out as `ValueShape` lets you verify exact compatibility against the hand-written type in a type test (the alias alone doesn't run a check).

Schemas compose — field schemas built up into larger ones are one cohesive concept and share a file. Keep schema code library-agnostic in principle; the pattern is the same across zod, valibot, and arktype.

## Namespace types alongside a class

When a class has useful accompanying types, export a type-only namespace of the same name. The namespace merges with the class, so `MyService` stays constructible while `MyService.Config` resolves as a type.

```ts
export declare namespace MyService {
  export type Config = { /* ... */ };
  export type InferSomething<T> = /* ... */;
}

export class MyService { /* ... */ }
```

## No type casts

**No type assertions (`as`, except `as const`) or non-null assertions (`!`) in normal code.** Use `satisfies`, type guards, or narrowing from `unknown` — `!` is a cast with different punctuation.

```ts
// ✅
if (error instanceof Error) log(error.message);

// ❌
log((error as Error).message);
log(error!.message);
```

A cast tells the compiler to stop checking; `satisfies` and guards keep it checking. When a cast feels necessary, the type is usually wrong somewhere upstream. In the rare low-level generic or overload implementation where TypeScript genuinely cannot verify a sound return type, isolate the narrowest possible assertion behind that helper boundary and explain why it holds.

`Object.entries()` is a common trap here — it widens keys to `string` even on a typed record, and the reflex is `as` at every call site. Prefer iterating a typed key collection (such as `Object.values(Status)`) instead; only wrap `Object.entries()` in a typed helper when runtime enumerable keys are guaranteed to match the declared key set.

### unknown over any

`any` is a cast that spreads: it disables checking for everything it touches. Use `unknown` and narrow — including for boundary APIs that return `any` (`res.json()`, `JSON.parse()`), which should be treated as `unknown` and validated or narrowed at the boundary instead of cast.

```ts
function parse(input: unknown) {} // ✅
function parse(input: any) {} // ❌
```

**Generic constraints are the exception.** Where a constraint has to accept any instantiation, `unknown` breaks variance (such as contravariant function parameters) and `any` is the working answer — keep `unknown` in covariant positions like return types when `any` isn't needed:

```ts
function formatCallback<T extends (...args: any[]) => unknown>(fn: T) {}
function formatData<T extends Data<any, any, any>>(data: T) {}
```

This comes up most in package code. `any` in a value position is still a smell.

## Generics

Name a type parameter when the name adds information. Don't prefix for its own sake.

```ts
type ValueOf<T> = T[keyof T];              // ✅ one universal param, T is clearest
type Registry<K, TManager> = /* ... */;    // ✅ K is obvious, TManager isn't
type ValueOf<TObject> = /* ... */;         // ❌ says nothing T didn't
```

`K` and `V` are fine where the role is conventional. Reach for a descriptive `TSomething` when the parameter stands for a domain concept a letter can't convey.

## Return types

**Let TypeScript infer.** An annotation that restates what inference already produces is a second source of truth that can drift.

Annotate when:

- the return type is a **discriminated union** — inference tends to collapse or widen the cases;
- **inference gets it wrong** or produces something unusable;
- it's **library code**, where an explicit signature is part of the contract and worth being in the reader's face;
- **`isolatedDeclarations` is enabled**, which requires it.

`isolatedDeclarations` is worth weighing rather than defaulting on: it forces explicit annotations on exported values whose types aren't trivially inferable, which fights derived-from-value patterns above (derived or spread enums, `as const satisfies` mappers, and inferred schemas) — the whole point there is that the type is derived from the value.

## Iteration

**`for...of` over `.forEach`.** `await`, `break`, `continue`, and `return` work as expected in a loop; in `.forEach`, `break`/`continue` aren't available, `return` only exits the callback, and returned promises are ignored — so a loop never has to be rewritten when the body grows.

```ts
// ✅
for (const user of users) {
 await sync(user);
 if (user.isArchived) continue;
}

// ❌ ignores returned promises, can't break; return only exits the callback
users.forEach(async (user) => {
 /* ... */
});
```

- Indexed `for (let i = 0; ...)` only when the index is actually used.
- `Object.entries()` for objects (or iterate a typed key collection / narrow typed wrapper when the runtime key set is closed).
- `.map` and `.filter` on their own are fine — they're expressions returning values.

**`flatMap` over a `filter`+`map` chain.** Return the value for a match, `[]` to skip:

```ts
// ✅
const names = users.flatMap((user) => (user.isActive ? user.name : []));

// ❌
const names = users.filter((user) => user.isActive).map((user) => user.name);
```

Return the bare value, not `[user.name]` — `flatMap` appends non-array returns directly, so wrapping just allocates a throwaway array. The exception is when the mapped value is itself an array (`user.tags`), which would get flattened a level; wrap that one.

A `flatMap` body can be as complex as it needs to be and is still preferable to chaining:

```ts
const names = users.flatMap((user) => {
 if (user.isActive) return user.name;
 if (user.otherThing) return user.fullName;

 return [];
});
```

## Control flow

**Prefer early returns over nesting.** Handle the exits first and let the main path sit unindented at the end.

```ts
// ✅
function process(user: User) {
 if (!user.isActive) return null;
 if (!user.hasAccess) return null;

 return buildProfile(user);
}

// ❌
function process(user: User) {
 if (user.isActive) {
  if (user.hasAccess) {
   return buildProfile(user);
  }
 }

 return null;
}
```

This applies anywhere a branch can exit — guard clauses in functions, `continue` in loops, and early `return` in a `flatMap` body.

## Exports

**Named exports only.** The exception is where a framework requires a default — Next.js pages and layouts, config files, and similar tool-owned files.

```jsonc
// e.g. eslint (or equivalent linter rule)
"import/no-default-export": "error"   // with an override for framework paths
```

## readonly

Apply `readonly` where it prevents a real bug or expresses an immutable contract, not as blanket ceremony:

- **Array and tuple parameters** get `readonly T[]` (or `readonly [A, B]` for tuples). A callee mutating a caller's array or tuple is a live bug and the annotation costs one word.
- `as const` already gives deep readonly on literals, mappers, and object-enums.
- **Don't force `readonly` on every object property or nested type.** Using `readonly` on object properties is fine when fields shouldn't be mutated (such as value objects, discriminants, or public contracts) — don't strip it where it's already used. Just don't require `readonly` or `Readonly<T>` everywhere when it becomes noisy or inconvenient; `Readonly<T>` is shallow, so deep immutability isn't something the type system supports cheaply.

`eslint-plugin-functional`'s `prefer-immutable-types` can enforce this broadly if the tradeoff ever changes.

## Required settings

Assume `strict: true`. Only what it doesn't already cover is listed here.

```jsonc
// tsconfig.json
{
 "compilerOptions": {
  "strict": true,
  "verbatimModuleSyntax": true,
  "noUncheckedIndexedAccess": true,
  "noImplicitOverride": true,
  "noFallthroughCasesInSwitch": true,
 },
}
```

`verbatimModuleSyntax` makes importing a type without an explicit `type` modifier a compiler error (TS1484), erases `type`-marked imports, and leaves value imports untouched in emitted output.

Keep type imports as top-level statements, separate from value imports:

```ts
import type { User } from "./types";
import { ProfileService } from "./services/profile";
```

```jsonc
// e.g. eslint (or equivalent linter rules)
"@typescript-eslint/consistent-type-imports": ["error", { "fixStyle": "separate-type-imports" }], // autofix style
"import/consistent-type-specifier-style": ["error", "prefer-top-level"]                           // enforces top-level statements
```

This mirrors `export type * from './types'` on the export side — type and value movement stay visually distinct in both directions.

`noUncheckedIndexedAccess` types `arr[0]` as `T | undefined`. It's the flag that gives the no-cast rule teeth, since the usual escape hatch is `arr[0]!`. Narrow it instead:

```ts
const first = users[0];
if (first === undefined) return;
// first is User here
```

`exactOptionalPropertyTypes` is **not required, but not discouraged either** — a per-package choice. It distinguishes an absent property from one explicitly set to `undefined`, which pays off well in packages where that difference carries meaning: patch and partial-update APIs, anything serializing to JSON, config merging. In packages without that distinction it mostly generates churn. Enable it where it earns its keep; don't add it uniformly and don't strip it where someone already has.
