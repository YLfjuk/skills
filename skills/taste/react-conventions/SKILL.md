---
name: react-conventions
description: Standards for writing React code — props, JSX, component structure, effects. Use when writing or reviewing any component or hook, even if conventions aren't mentioned. Targets React 19.
---

# React Conventions

Standards for how React is written here.

**Precedence.** Existing standards supersede this guide — highest priority first: mandatory organization policies, the repo's own standards, the framework's, and the language's. Refactor existing code toward these standards only when asked to.

## Props

Name the props type `Props`:

```tsx
type Props = {
  title: string;
};
```

If the type must be exported, or the file contains multiple components (prefer one per file, except for compound sub-components or tiny module-scope helpers), name it for its component: `CardProps`.

When a component takes children, use `PropsWithChildren`:

```tsx
import type { PropsWithChildren } from "react";

type Props = { title: string };

export function Card(props: PropsWithChildren<Props>) {
  /* ... */
}
```

`PropsWithChildren` makes `children` optional. When callers must provide children, declare `children: ReactNode` directly on `Props` instead (note that `ReactNode` still permits `null` and `undefined`).

**Do not use `React.FC`.** It can't type a generic component directly, forces a variable annotation over a function declaration, and adds noise over plain typed `props` with inferred return types.

### Destructure in the body, not the signature

```tsx
// ✅ Preferred
export function Card(props: PropsWithChildren<Props>) {
  const { title, children } = props;

  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// ⚠️ Acceptable for very simple components
export function Card({ title, children }: PropsWithChildren<Props>) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

Body destructuring is clearer when debugging and makes it easy to add defaults, logging, or a spread without reshaping the signature.

### Destructure order

1. Variables — `{ isEnabled }`
2. Variables with defaults — `{ isEnabled = false }`
3. Functions — `{ onClick }`
4. Functions with defaults — `{ onClose = noop }`
5. Ref — `{ ref }`
6. Children — `{ children }`
7. Rest — `{ ...rest }`

```tsx
export function Button(props: Props) {
  const { title, isDisabled = false, onClick, onClose = noop, ref, children, ...rest } = props;

  /* ... */
}
```

## JSX prop order

1. `key`
2. `ref`
3. Truthy booleans, shorthand — `isEnabled`, never `isEnabled={true}`
4. Constants and literals — `prop={"value"}`, `prop={42}`
5. Variables and state — `prop={variable}`
6. Functions and handlers — `onClick={handleClick}`

```tsx
<Component key={key} ref={ref} isEnabled prop={"value"} label={label} onClick={handleClick} />
```

Place `{...rest}` by override semantics (before explicit props that should override it, or after defaults) rather than in a fixed band.

**Braces on string props**, including literals: `prop={"value"}`, not `prop="value"`. Every other value needs the braces anyway, so this keeps one shape at every call site. This applies to props only — plain text children stay unwrapped: `<div>text</div>`.

Some linter configs strip curly braces from string props on `--fix` (and `jsx-curly-brace-presence` defaults to `"never"`). Configure it to match:

```jsonc
// e.g. eslint (or equivalent linter rule)
"react/jsx-curly-brace-presence": ["error", { "props": "always", "children": "never" }]
```

**Note the two orders differ deliberately.** `ref` is second at the call site and fifth in the destructure. At the call site `key` and `ref` are identity and meta, so they surface where a reader scans first; in the body they're passthrough plumbing you rarely touch, so they sink next to `children` and `...rest`.

## Component declaration

Use **function declarations**. Arrow functions are for small one-liner components. Never define a component inside another component's render body — each render creates a new component identity, causing React to unmount and remount its subtree and lose state.

```tsx
// ✅
export function Card(props: Props) {
  /* ... */
}

// ✅ one-liner
const Divider = () => <hr />;
```

### Compound components

Use the compound pattern for elements meant to be used together:

```tsx
export function Dropdown(props: PropsWithChildren) {
  const { children } = props;

  return <div>{children}</div>;
}

Dropdown.Item = DropdownItem;
```

The sub-component can live in the same file (at module scope) or be imported from its own file — `./dropdown-item.tsx`, kebab-case like any other file.

## Component body order

1. **Props destructure**
2. **Hooks and constants** the hooks or effects depend on
3. **Effects**
4. **Early returns**
5. **Other constants** — derived values and handlers used only by the JSX
6. **JSX**

```tsx
export function UserPanel(props: Props) {
  const { userId, onSelect } = props;

  const [isOpen, setIsOpen] = useState(false);
  const { data, isLoading } = useUser(userId);
  const containerRef = useRef<HTMLDivElement>(null);
  const sortedItems = useMemo(() => (data ? sort(data.items) : []), [data]);

  useEffect(() => {
    /* ... */
  }, [userId]);

  if (isLoading) return <Spinner />;
  if (!data) return null;

  const label = formatLabel(data.name, sortedItems);

  function handleSelect() {
    setIsOpen(false);
    onSelect?.();
  }

  return (
    <div ref={containerRef}>
      <button aria-expanded={isOpen} onClick={handleSelect}>
        {label}
      </button>
    </div>
  );
}
```

**What separates band 2 from band 5:** all hooks (and anything a hook or effect needs) go above the early returns; plain derived values and handlers used only by the JSX go after the early returns, where values are already narrowed and early exits skip unnecessary work.

Within band 2, prefer state and data hooks, then `useRef` → `useMemo` → `useCallback`. This is a soft preference — dependencies between hooks, or on non-hook values, override it.

Early returns over a top-level ternary, and over nesting generally.

## Event handlers

**`onX` for props, `handleX` for internal wrappers.** A `handleX` wrapper should do something beyond forwarding — its name tells the reader there's extra logic to find. Pass a prop straight through when signatures match; wrapping to adapt arguments or drop a DOM event counts as real work.

```tsx
// ❌ pointless wrapper
function handleClick(event: MouseEvent<HTMLButtonElement>) {
  onClick(event);
}
<button onClick={handleClick} />

// ✅ pass it straight through
<button onClick={onClick} />

// ✅ the wrapper earns its name
function handleClick() {
  setIsOpen(false);
  onClick?.();
}
<button onClick={handleClick} />
```

Call an optional handler with `onClick?.()`. When the destructure supplied a default (`onClose = noop`), call it directly.

## Hooks and effects

Follow React's [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect) — **avoid unnecessary effects.**

When an effect needs to read the latest props, state, or callbacks without re-running when they change, extract that genuinely non-reactive logic with [`useEffectEvent`](https://react.dev/reference/react/useEffectEvent) (React 19.2+). Keep Effect Events out of dependency arrays, call them only from effects, and don't use them to hide real reactive dependencies.

Place effects after all other hooks, per the body order above.

Custom hooks return an object once there are more than two values (a bare value for one; a tuple or object for two).

When a hook takes an options object, name the type `Options`, paralleling `Props` (or `UseUserOptions` if it must be exported):

```ts
type Options = { userId: string; isEnabled?: boolean };

export function useUser(options: Options) {
  /* ... */
}
```

A hook taking a single bare value needs no named `Options` type — don't wrap one argument in an object to have somewhere to put the name.

## Keys

Don't use an array index as `key`. React reuses the wrong component instance on reorder, insert, or delete, which corrupts state silently.

```tsx
// ✅
{items.map((item) => <Row key={item.id} />)}

// ❌
{items.map((item, index) => <Row key={index} />)}
```

Enforce this with a linter rule (e.g., `react/no-array-index-key` in ESLint). For a constant, predefined array that never reorders, prefer a unique primitive value itself (`key={step}`) when available; otherwise keep the rule enabled and disable it inline rather than relying on judgment.

## Older React versions

React 19 is the target. On older versions, two things change:

- **React 18 — `ref` is not a prop.** Components that accept a `ref` use `forwardRef`, taking `ref` as the second argument. Such a component is a `const` wrapping a named function expression (preserving DevTools `displayName`), and `ref` drops out of the props destructure order.

  ```tsx
  export const Input = forwardRef<HTMLInputElement, Props>(function Input(props, ref) {
    const { value, onChange, ...rest } = props;

    return <input ref={ref} {...rest} value={value} onChange={onChange} />;
  });
  ```

- **Before React 19.2 (18 and 19.0–19.1) — `useEffectEvent` isn't available yet.** Sync the latest callback into a ref inside `useLayoutEffect` (or `useEffect`) rather than mutating `ref.current` during render.

Everything else applies unchanged.
