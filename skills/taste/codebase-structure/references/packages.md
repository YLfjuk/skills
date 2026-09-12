# Packages, Boundaries, and Public APIs

- [Boundaries vs layers](#boundaries-vs-layers)
- [App and package source roots](#app-and-package-source-roots)
- [Application infrastructure](#application-infrastructure)
- [When to extract a package](#when-to-extract-a-package)
- [Partial extraction](#partial-extraction)
- [Public APIs](#public-apis)
- [Submodules](#submodules)
- [Naming](#naming)

---

## Boundaries vs layers

Package boundaries and conceptual layers are **independent dimensions**:

1. Conceptual dependency direction — how responsibilities depend on each other.
2. Package/app boundaries — where those responsibilities physically live and are independently consumed.

A layer needn't map to a package, and a package needn't hold only one layer. All of these are valid for the same architecture:

```text
packages/sdk/src/{core,features,domains,design}/       # many layers, one package
packages/core/  packages/users/  packages/dashboard/   # one layer each
apps/web/src/features/users/                           # inside an app
```

**Moving code into a package does not change its conceptual layer.** A package is not a flat bag of modules: organize its `src/` by the same responsibilities as an app. It may contain features, and it may contain domains when it composes those features for a specific context.

---

## App and package source roots

```text
apps/{app}/src/
packages/{package}/src/
```

The repo is always a monorepo, but only the roots in use exist — a library monorepo has no `apps/`. See SKILL.md. `src/` beneath them is an organizational convention, not a layer. Conceptual layers may sit directly beneath either root, and one package may hold several responsibilities.

Do not create package boundaries just to make the physical structure mirror the conceptual architecture.

---

## Application infrastructure

An app may hold infrastructure belonging to the app itself:

```text
apps/web/src/configs/
```

Keep app infrastructure separate from feature/domain code where that clarifies ownership. App-specific environment configuration belongs to the app's configuration system, not to whichever feature happens to consume it.

---

## When to extract a package

Extract when the boundary itself provides a benefit: reuse, isolation, ownership, distribution, dependency management, or an independently consumable API.

Not for symmetry with the layer diagram, and not for hypothetical future reuse.

Test the boundary with this question:

> If this layer and everything less specific than it were extracted into packages, would that make the next more-specific layer significantly harder to compose, maintain, or depend on?

A good boundary lets more-specific layers keep working with little more than changed imports or package dependencies.

---

## Partial extraction

Extraction doesn't require taking a whole layer. A feature can split along how independently its parts can be consumed:

```text
features/users/{types,utils,services,components}/
```

becomes

```text
packages/users/src/{types,utils,services}/     # reusable
apps/web/src/features/users/components/        # app-specific, stays
```

---

## Public APIs

A package or designated feature entry point may expose a curated API:

```text
packages/users/src/
  components/
  services/
  types/
  index.ts
```

```ts
export type * from "./types";
export { createUserService } from "./services/create-user-service";
```

Internal modules are not public merely because they're physically reachable. Consumers use the supported entry point:

```ts
import { X } from "@scope/users"; // ✓
import { X } from "@scope/users/src/internal/X"; // ✗
```

Keeping this boundary meaningful is what makes later restructuring or extraction cheap.

---

## Submodules

A package may expose independently consumable functionality through submodules when the extra code enriches the core behavior:

```text
@scope/feature
@scope/feature/react
@scope/feature/vue
```

Or as separate packages when sufficiently independent:

```text
@scope/feature
@scope/feature-react
```

Appropriate for framework-specific bindings, optional integrations, optional peer dependencies, and independently consumable functionality.

Do not force optional or framework-specific code into the primary entry point — it imposes unnecessary dependencies on every consumer.

### Entry points are public surfaces, not implementation layers

`.` and a submodule entry point such as `./node` define what consumers can import. They may re-export a curated API or perform only the minimal composition needed needed to expose it; they do not become a general home for modules merely because those modules share an export path.

Put implementation with the feature, domain, or other unit that owns it. Subexports and layers are separate dimensions: a subexport may expose a domain, a feature, or a curated API spanning several units. A domain will often have a corresponding subexport because its boundary is independently useful to consumers, but neither the import path nor the layer determines the other. Keep the public path stable while organizing implementation by ownership.

---

## Naming

```text
@{scope}/{name}
@{scope}/{sub-purpose}.{name}
```

```text
@scope/web
@scope/feature-react
@scope/tests.utils
```

A scoped namespace can separate registries, useful when some packages publish publicly and others stay in a private org registry:

```text
@{scope}.{sub-scope}/{name}
```

Otherwise, stay consistent with the rest of the repo or organization.

---

## Promotion path

Code should begin at the narrowest reasonable scope and move outward only for a concrete reason. A typical progression:

```text
core/components/breadcrumb.tsx
  → foundation/breadcrumbs/{component.tsx,types.ts,utils.ts}    # grew into an app-level block
  → packages/users/                                              # needed by multiple apps
```

Each step should be triggered by something real — genuine reuse, a clearer owner, dependency isolation, independent distribution, encapsulation, a meaningful boundary. Not by anticipation.
