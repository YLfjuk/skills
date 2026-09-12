# Layer Responsibilities

Read the section for the layer in question. The hard distinctions are at the bottom.

- [Core](#core)
- [UI](#ui)
- [Lib](#lib)
- [Design](#design)
- [Foundation](#foundation)
- [Features](#features)
- [Domains](#domains)
- [Application composition (frontend, backend, other app types)](#application-composition)
- [Colocation](#colocation)
- [Distinctions that are easy to get wrong](#distinctions-that-are-easy-to-get-wrong)

Any of these layers may be omitted, combined with another, split across directories, or spread across packages. Their names describe responsibilities, not a required directory listing.

---

## Core

App-wide, low-level, generic code with **no business or feature awareness**.

```text
core/
  components/
  utils/
    is-defined.ts
  types/
```

Holds generic utilities and types, low-level logical components, generic guards, small reusable primitives, application-independent helpers.

Core must not depend on business domains or feature-specific concepts. If a piece of core code needs to know what a user is, it isn't core.

---

## UI

Base UI primitives and UI infrastructure — particularly vendor- or framework-derived components.

```text
ui/
  components/
    button.tsx
    dialog.tsx
    input.tsx
  hooks/
```

The structure here may stay close to the upstream or generated shape (e.g. whatever shadcn/ui emits) rather than being reorganized to fit this guide. The purpose is stable low-level building blocks, not application-specific visual composition.

Omit this layer entirely when the app has no UI.

---

## Lib

Bindings, adapters, integrations, and reusable orchestration around external libraries or technologies.

```text
lib/
  auth/       # better-auth
  form/       # tanstack-form
  router/     # tanstack-router
  query/      # react-query
```

`lib` owns reusable third-party coupling and provides app-friendly abstractions over external libraries.

**This does not mean all use of a library lives here.** Generic integration goes in `lib`; application-specific usage goes with the feature or domain that owns the behavior:

```text
lib/query/                 # query client setup, generic utilities
features/users/api/
  queries/                 # app-specific react-query usage
  mutations/
```

---

## Design

Application-owned design-system components and higher-level visual building blocks. Sits above base UI primitives; may build on `ui` and on library integrations.

```text
design/
  components/
    page-header/
    empty-state/
  layout/
```

More complex visual blocks adapted from component registries belong here when they form part of the app's reusable design language. `design` should stay independent of business and domain concepts.

May be merged with `ui`, kept separate, or omitted.

### UI layers must not accumulate business logic

`ui/button.tsx` stays a generic primitive. A design-system component may hold presentation and reusable interaction behavior, but must not become the owner of domain behavior just because a UI element triggers it.

When a `ui` or `design` component seems to need business logic, there are two moves — often both:

**1. Split the component.** Separate the generic part from the business-aware part and keep only the generic part in `ui`/`design`. A `UserAvatarMenu` that loads the current user and handles sign-out splits into a presentational `AvatarMenu` in `design` — taking a label, an image, and menu items as props — and nothing else.

**2. Compose the capability at the call site.** The feature or domain that owns the behavior renders the generic component and supplies the behavior:

```tsx
// features/auth/components/user-menu.tsx
<AvatarMenu label={user.displayName} items={[{ label: "Sign out", onSelect: signOut }]} />
```

The test for whether you got it right is the import direction: **the feature imports the design component, never the reverse.** If `design/` needs to import from `features/` or `domains/`, the split hasn't happened yet.

This applies to data as much as to actions — a design component that fetches, or that knows the shape of an API response, has taken on business awareness. Pass it what it needs to render.

---

## Foundation

Reusable **application-level** building blocks that belong to no particular business domain or feature. More substantial and more app-oriented than `core` primitives, but not business-specific.

```text
foundation/
  breadcrumbs/
    components/
    utils/
    types/
```

Covers reusable app-level UI or behavioral blocks, cross-feature building blocks, generic application workflows, reusable infrastructure with application semantics but no domain ownership.

The difference from `core` is abstraction level: `core` is generic low-level primitives; `foundation` is reusable application-level blocks.

---

## Features

Application **capabilities** — contracts and behavior. A feature is what the app can _do_, independent of the context it's used in.

```text
features/
  users/
    components/
    services/
    utils/
    types/
```

May contain feature-level APIs and contracts, feature-specific components and utilities, types, and whatever else the capability requires.

A feature may be composed differently by multiple domains, so feature code should avoid depending on a specific domain or app context when the capability is meant to be reusable.

### Cross-feature dependencies

Allowed when the dependency is a legitimate capability relationship — `features/users → features/auth` is fine if user behavior genuinely needs authentication.

Avoid bidirectional coupling (`features/users ↔ features/auth`). When two features become tightly coupled, consider whether one should own the shared behavior, a lower-level responsibility should be extracted, the capabilities should merge, or the composition actually belongs in a domain.

The goal isn't to forbid feature-to-feature edges — it's to keep the feature graph understandable.

---

## Domains

Concrete, **context-specific** implementations and compositions of features. A domain is a business or application context: a dashboard, a demo, an admin area, another distinct environment.

```text
domains/
  dashboard/
    components/
    utils/
    types/
    features/
      users/
      auth/
```

Domains may depend on and compose many features. Different domains may compose the same feature differently. A runtime can also be a domain when it is a distinct context that supplies or composes capabilities differently, rather than merely an incidental platform dependency:

```text
domains/dashboard/features/users/
domains/demo/features/users/
domains/node/
domains/browser/
```

```text
feature = capability
domain  = contextual implementation / composition
```

A feature must not depend on a domain that consumes it.

### Organizing inside a feature or domain

Organize around the responsibility owned, not arbitrary technical categories. `components`, `services`, `utils`, `types` are organizational tools — add one when there's enough related code to warrant the structure.

---

## Application composition

Every app composes and wires existing capabilities at its outermost layer. **Which layers do that work depends on the kind of app**, and no app type is the baseline: a frontend and a backend are peer app types, each introducing the composition layers its runtime calls for. A CLI app might have `commands`; a worker might have `jobs` or `consumers`.

What unites all of them: **they compose and wire existing capabilities.** They must not become a general-purpose home for reusable feature or domain logic. Business rules, validation, and persistence behavior belong to the feature or domain being called. When a composition layer grows logic worth testing on its own, that logic has an owner elsewhere.

### Frontend apps

```text
apps/web/src/
  features/
  layouts/
  pages/
  routes/
```

**Layouts** — shared structural composition. May compose domains, features, design components, and lower-level code.

```text
layouts/
  dashboard/
  authenticated/
  settings/
```

**Pages** — route-level UI and a screen's general structure. Compose features, domains, layouts, and lower-level presentation; don't host reusable business logic. Pages play the same application-composition role as backend controllers: they assemble existing capabilities for one user-facing outcome.

```text
pages/
  users/
  settings/
  dashboard/
```

**Routes** — own routing logic: route definitions, loading and validation, redirects, guards, and route-specific orchestration. In file-based routing, they also contain the framework-required entry module. That convention does not make the route module the default owner of a screen's general structure or composition.

Keep a trivial route module intact when extracting a page would add indirection without a boundary. When the screen's structure or composition becomes substantial, put it in `pages/` and let the route handle the routing work around it. Follow a framework or repository convention that deliberately combines the two.

### Backend apps

```text
apps/api/src/
  features/
  middlewares/
  controllers/
  routes/
```

**Controllers** — receive a request, call into features and domains, shape a response. The same constraint as pages: a controller composes capabilities, it does not own them.

**Middlewares** — cross-cutting request concerns such as auth checks, logging, and error shaping. A middleware wires behavior into the request pipeline; the behavior itself belongs to whatever owns it. An auth middleware calls the auth capability, it doesn't implement it.

**Routes** — wire URLs to controllers and middlewares.

### Other app types

The pattern generalizes. Identify what the runtime requires as an entry point, and treat those as the app's composition layers — subject to the same rule that they wire rather than own. Don't force a CLI or worker app into frontend or backend vocabulary.

A composition layer holds the units named by that layer; the application entry point assembles those units into the running program. For example, commands belong in `commands/`, while the program that selects and runs commands belongs at the CLI entry point. Apply the same distinction wherever a runtime has both named units and a bootstrapping entry point.

### Watch the word "services"

Backend code often calls its business layer `services/`, but here `services/` is a technical subdirectory _inside_ a feature (`features/users/services/`) — owned by the feature, not a top-level composition layer. Don't create `apps/api/src/services/` as a general home for business logic; that's the feature layer wearing a different name.

---

## Colocation

Colocation means keeping code within the directory or conceptual unit owned by the same responsibility; it does not require keeping every distinct concept in the same file. A unit may have several focused files without becoming shared code.

Colocate code with the narrowest responsibility that owns it:

```text
features/users/api/queries/get-user.ts
```

is better than putting user-specific behavior in a global utility directory just because it happens to be a function.

Conversely, genuinely reusable code shouldn't be duplicated or artificially bound to one feature.

> Keep code close to its owner until it has a legitimate reason to become shared.

---

## Distinctions that are easy to get wrong

**`core` vs `foundation`** — abstraction level, not subject matter. `is-defined.ts` is core. A breadcrumbs system with components, types, and utils is foundation. Ask: is this a primitive, or an application-level block?

**`feature` vs `domain`** — capability vs context. "Managing users" is a feature. "The admin dashboard, which manages users this particular way" is a domain. If it describes _what_ the app does, it's a feature; if it describes _where or for whom_, it's a domain.

**`ui` vs `design`** — provenance and level. Vendor/generated primitives are `ui`. Blocks you own that express the app's visual language are `design`.

**`lib` vs feature-owned integration** — generic setup and adapters are `lib`; specific calls that encode app behavior belong to the feature or domain making them.

**Any layer vs "it's shared now"** — multiple consumers do not make domain logic generic. A user-specific rule called from three places is still user-specific; it belongs to `features/users` with the callers importing it.
