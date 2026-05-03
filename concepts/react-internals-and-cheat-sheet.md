# React Internals & Cheat Sheet (React 19)

> A working architect's reference — every API I reach for in day-to-day work, plus enough internals to reason about *why* React behaves the way it does. Up to date with React 19.

If you write React long enough, you stop thinking in components and start thinking in renders, commits, and lanes. This guide is the cheat sheet I keep close — hooks, patterns, the React 19 surface — followed by a short internals tour so the cheat sheet isn't a black box.

---

## Why this matters

React doesn't reward memorization — it rewards a model of *when* things run and *why*. Two components that look identical can behave completely differently because one captures a stale closure, mutates props, or fights the reconciler. The cheat sheet gets the API right; the internals get the *behavior* right.

In practice: most React bugs trace back to one of three things — a wrong dependency array, a key that doesn't actually identify the row, or a state update batched differently than the developer assumed.

---

## Mindset

- **Render is a description; commit is the side effect.** Your function returns JSX; React decides what to do with it. Treat render functions as pure.
- **Identity matters more than equality.** Reference equality drives hook re-runs, child re-renders, and list reuse — not deep equality.
- **Effects are escape hatches, not glue.** If a value can be *derived* from props/state, derive it. If it belongs to a user event, put it in the handler. Reach for `useEffect` only to synchronize with systems React doesn't own.
- **Lift state where it belongs, not higher.** State that lives too high re-renders the world; state that lives too low can't be shared.
- **In React 19, the compiler memoizes for you.** Stop reflexively wrapping things in `useMemo`/`useCallback` — write the obvious code and let the compiler earn its keep.
- **Suspense is a rendering primitive, not a loading library.** It's how the tree pauses while data resolves; everything else (skeletons, spinners, transitions) is built on top.

---

## The cheat sheet

### Components

```tsx
// Function component — the only kind worth writing in 2026.
function Greeting({ name }: { name: string }) {
  return <h1>Hello, {name}</h1>;
}

// Default export when the file *is* the component; named export otherwise.
export default Greeting;
```

Worth knowing: class components still work but receive almost no new features. Hooks, Suspense ergonomics, the compiler, and most third-party libs assume function components.

### JSX rules in one breath

- One root element (or a `Fragment` / `<>...</>`).
- `className`, `htmlFor`, camelCase event handlers (`onClick`, not `onclick`).
- Expressions go in `{}`. Strings, numbers, arrays render; objects, booleans, `null`, `undefined` don't.
- Lists need stable, unique `key` props — see *Reconciliation* below.
- React 19 lets you render `<title>`, `<meta>`, `<link>`, `<script>` anywhere; React hoists them into `<head>`.

### Built-in hooks — quick table

| Hook | What it's for | Key gotcha |
|---|---|---|
| `useState` | Local state | Setter is stable; pass updater fn for sequential updates |
| `useReducer` | Local state with structured transitions | Reducer must be pure |
| `useEffect` | Sync with external system | Runs after commit, async; cleanup runs before next effect |
| `useLayoutEffect` | Read DOM / mutate before paint | Blocks paint — use sparingly |
| `useInsertionEffect` | CSS-in-JS libs only | Almost never user code |
| `useRef` | Mutable box that survives renders | Mutating doesn't trigger re-render |
| `useImperativeHandle` | Customize what a `ref` exposes | Pair with forwardRef pre-19; in 19 just accept `ref` as prop |
| `useMemo` | Cache expensive value | Compiler often makes this redundant in 19 |
| `useCallback` | Stable fn identity | Same — let the compiler do it |
| `useContext` | Read context value | Re-renders every consumer when value identity changes |
| `useId` | Stable SSR-safe ID | Don't use as a key |
| `useTransition` | Mark state update as non-urgent | Returns `[isPending, startTransition]` |
| `useDeferredValue` | Lag a value behind during heavy work | The "let the typing stay snappy" hook |
| `useSyncExternalStore` | Subscribe to external store | The official store-binding hook (Zustand, Redux, etc.) |
| `useDebugValue` | Label custom hooks in DevTools | Library code only |
| `use` *(19)* | Unwrap a Promise or Context inside render | Can be called conditionally — unique among hooks |
| `useActionState` *(19)* | Form-action state + pending | Replaces `useFormState` |
| `useOptimistic` *(19)* | Optimistic UI during a transition | Pairs with Server Actions |
| `useFormStatus` *(19)* | Read parent `<form>`'s pending state | For nested submit buttons |

### `useState` patterns

```tsx
// Lazy initial state — the function only runs on mount.
const [tree, setTree] = useState(() => buildHugeTree());

// Functional updater — required when the next value depends on the previous.
setCount(c => c + 1);

// Object state — replace, don't mutate.
setUser(u => ({ ...u, name: "Ada" }));
```

A caveat: `useState` does a `Object.is` check before re-rendering. Returning the same reference bails out — but a new object that looks identical does not.

### `useReducer` when state has structure

```tsx
type State = { items: Item[]; filter: string };
type Action =
  | { type: "add"; item: Item }
  | { type: "remove"; id: string }
  | { type: "filter"; filter: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "add": return { ...state, items: [...state.items, action.item] };
    case "remove": return { ...state, items: state.items.filter(i => i.id !== action.id) };
    case "filter": return { ...state, filter: action.filter };
  }
}

const [state, dispatch] = useReducer(reducer, { items: [], filter: "" });
```

In practice: I reach for `useReducer` the moment two pieces of state need to change together, or when a component has more than ~3 `useState` calls.

### `useEffect` rules of thumb

```tsx
useEffect(() => {
  const sub = source.subscribe(setData);
  return () => sub.unsubscribe();
}, [source]);
```

- Effects run *after* commit, asynchronously.
- The cleanup runs before the next effect *and* on unmount.
- Strict Mode in dev double-invokes effects to surface missing cleanup. This is a feature, not a bug.
- `[]` deps = run once on mount. No deps = run every render. Don't lie about deps to silence the linter.

A reframe: if a `useEffect` is just calling `setState` based on props, you almost certainly want a derived value or `useMemo`, not an effect.

### `useRef` — two distinct uses

```tsx
// 1. DOM access
const inputRef = useRef<HTMLInputElement>(null);
useEffect(() => inputRef.current?.focus(), []);

// 2. Mutable box that doesn't trigger renders
const renderCount = useRef(0);
renderCount.current++;
```

### `useTransition` & `useDeferredValue`

```tsx
const [isPending, startTransition] = useTransition();

function onSearch(q: string) {
  setQuery(q);                          // urgent — keeps input snappy
  startTransition(() => setResults(q)); // non-urgent — interruptible
}
```

`useDeferredValue` is the receiving end:

```tsx
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => search(deferredQuery), [deferredQuery]);
```

### Context

```tsx
// React 19: the context itself is the provider.
const Theme = createContext<"light" | "dark">("light");

<Theme value="dark">
  <App />
</Theme>;

// Pre-19: <Theme.Provider value="dark">
```

Worth knowing: every consumer re-renders when the context value's *reference* changes. Memoize the value, or split context into "rarely changes" + "frequently changes" pairs.

### Suspense & Error Boundaries

```tsx
<ErrorBoundary fallback={<Error />}>
  <Suspense fallback={<Skeleton />}>
    <Profile id={id} />
  </Suspense>
</ErrorBoundary>
```

- `Suspense` catches *thrown promises* (from `use`, RSC, lazy components, data libs).
- `ErrorBoundary` catches *thrown errors* (still class-component-only — every library ships one).
- Pair them at every async leaf you want to isolate.

### Code splitting

```tsx
const Chart = lazy(() => import("./Chart"));

<Suspense fallback={<Skeleton />}>
  <Chart />
</Suspense>;
```

### `memo`, `useMemo`, `useCallback` — the pre-compiler trio

```tsx
const Row = memo(function Row({ item }: { item: Item }) { ... });

const total = useMemo(() => items.reduce(sum, 0), [items]);
const onClick = useCallback(() => doStuff(id), [id]);
```

A caveat: in React 19 with the compiler enabled, most of these become noise. Keep `memo` for true list-row hot paths and let the compiler handle the rest. Don't pre-emptively memoize.

---

## React 19, the new surface

### Actions — async transitions, first-class

Any async function passed to `startTransition`, a `<form action={...}>`, or `formAction` is an **Action**. React tracks pending state, errors, and optimistic updates around it.

```tsx
async function updateName(formData: FormData) {
  await api.updateName(formData.get("name"));
}

<form action={updateName}>
  <input name="name" />
  <SubmitButton />
</form>;
```

### `useActionState` — state that flows through an Action

```tsx
const [state, formAction, isPending] = useActionState(
  async (prev, formData) => {
    try {
      await api.save(formData);
      return { ok: true };
    } catch (e) {
      return { ok: false, error: String(e) };
    }
  },
  { ok: false }
);

<form action={formAction}>...</form>;
```

### `useFormStatus` — nested submit knows its parent

```tsx
function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? "Saving..." : "Save"}</button>;
}
```

### `useOptimistic` — predict, then reconcile

```tsx
const [optimisticTodos, addOptimistic] = useOptimistic(
  todos,
  (state, newTodo: Todo) => [...state, { ...newTodo, pending: true }]
);

async function add(formData: FormData) {
  const todo = { id: crypto.randomUUID(), text: formData.get("text") };
  addOptimistic(todo);
  await api.create(todo);
}
```

### `use` — the promise-and-context unwrapper

```tsx
function Profile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);   // suspends until resolved
  const theme = use(ThemeContext); // also reads context
  return <h1 style={{ color: theme.fg }}>{user.name}</h1>;
}
```

`use` is the only hook that can be called conditionally — inside an `if`, inside a loop. It's how React 19 lets you read promises without a separate library.

### `ref` as a prop — `forwardRef` retired

```tsx
// React 19
function Input({ ref, ...props }: { ref?: Ref<HTMLInputElement> } & InputHTMLAttributes<HTMLInputElement>) {
  return <input ref={ref} {...props} />;
}

// Pre-19
const Input = forwardRef<HTMLInputElement, Props>((props, ref) => <input ref={ref} {...props} />);
```

### Document metadata hoisting

```tsx
function Article({ post }: { post: Post }) {
  return (
    <>
      <title>{post.title}</title>
      <meta name="description" content={post.excerpt} />
      <link rel="canonical" href={post.url} />
      <article>{post.body}</article>
    </>
  );
}
```

React lifts these into `<head>` at commit time — works with SSR, RSC, and CSR.

### Asset preloading

```tsx
import { preload, preinit, prefetchDNS, preconnect } from "react-dom";

preload("/fonts/inter.woff2", { as: "font", crossOrigin: "" });
preinit("/styles/critical.css", { as: "style" });
prefetchDNS("https://api.example.com");
preconnect("https://cdn.example.com");
```

### Server Components & Server Actions

```tsx
// app/page.tsx — Server Component (default in app router)
export default async function Page() {
  const posts = await db.posts.findMany();
  return <PostList posts={posts} />;
}

// app/actions.ts
"use server";
export async function createPost(formData: FormData) {
  await db.posts.create({ data: { title: formData.get("title") } });
  revalidatePath("/");
}

// Form anywhere
<form action={createPost}>...</form>;
```

In practice: RSC moves data fetching to the server, ships zero JS for server-only components, and lets you call server functions directly from forms. The mental shift is bigger than the API.

### React Compiler

Babel/SWC plugin that auto-memoizes by analyzing component dataflow. Enable per-project; remove `useMemo`/`useCallback` calls the compiler can prove are equivalent. As of 19 it's stable enough for production at most teams; check the [compiler docs](https://react.dev/learn/react-compiler) for the current eslint rule set.

---

## Patterns that hold up

### Derive, don't sync

```tsx
// ❌ effect-as-glue
const [fullName, setFullName] = useState("");
useEffect(() => setFullName(`${first} ${last}`), [first, last]);

// ✅ derive
const fullName = `${first} ${last}`;
```

### Lift state, but only one level

Push state down until siblings stop sharing it; lift it up the moment they need to.

### Reset state by changing `key`

```tsx
<Profile key={userId} userId={userId} />
```

When `key` changes, React unmounts and remounts — the cleanest way to reset all internal state.

### Compound components

```tsx
<Tabs defaultValue="overview">
  <Tabs.List>
    <Tabs.Trigger value="overview">Overview</Tabs.Trigger>
    <Tabs.Trigger value="details">Details</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel value="overview">...</Tabs.Panel>
  <Tabs.Panel value="details">...</Tabs.Panel>
</Tabs>
```

Built with context — children find their parent's state without prop-drilling.

### Render props / function-as-children

Less common since hooks, but still the cleanest answer for "give me data, I'll render it" libraries (e.g. virtualization, drag-and-drop).

### Custom hooks for cross-cutting state

```tsx
function useDebouncedValue<T>(value: T, ms: number): T {
  const [v, setV] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setV(value), ms);
    return () => clearTimeout(id);
  }, [value, ms]);
  return v;
}
```

A hook is just a function that uses other hooks. Naming convention `useX` is what the linter keys off.

---

## Under the hood

### Fiber — what a "component instance" actually is

A **fiber** is a plain JS object that represents one unit of work — one element in the tree. Each fiber has pointers to its parent, first child, and next sibling, forming a linked tree (cheaper to traverse and resumable, unlike a recursive call stack).

```
FiberRoot
  └─ HostRoot fiber
       └─ App fiber
            ├─ Header fiber
            └─ Main fiber
                 └─ List fiber
                      └─ Row fibers...
```

Each fiber stores:

- `type` — function/class/host element
- `stateNode` — the actual DOM node or component instance
- `pendingProps` / `memoizedProps`
- `memoizedState` — head of the hooks linked list
- `flags` — what to do at commit (placement, update, deletion)
- `lanes` — which priority lanes have pending work

### Two trees — current vs work-in-progress

React keeps two fiber trees and **double-buffers** between them.

- **current** — what's on screen.
- **workInProgress** — being built during render.

When render finishes, React swaps `current = workInProgress` in one atomic step. If the render is interrupted or thrown away, the on-screen tree is untouched.

### Render phase vs commit phase

- **Render phase** — pure, *interruptible*. React walks fibers, runs your function components, builds the WIP tree. Can be paused, resumed, even thrown away.
- **Commit phase** — synchronous, *uninterruptible*. React applies DOM mutations, runs `useLayoutEffect`, then flushes `useEffect` asynchronously.

Lesson learned: anything you do in render runs *more than once* — possibly with stale results. Side effects belong in commit (via effects) or in event handlers.

### Reconciliation — diffing rules

When React compares old and new children, the rules are simple by design (so they're O(n) instead of O(n³)):

1. **Different element type → throw away the subtree.** `<div>` becoming `<span>` unmounts and remounts everything below.
2. **Same type → reuse the fiber, update props.**
3. **Lists → compare by `key`.** No key (or index keys) means React matches by position — fine for static lists, broken for reorders/inserts.

```tsx
// ❌ key-as-index breaks on reorder
items.map((item, i) => <Row key={i} item={item} />)

// ✅ stable, unique key
items.map(item => <Row key={item.id} item={item} />)
```

### Lanes & the Scheduler

React 17+ replaced the older expiration-time priority with a **lane** model — a 31-bit bitmask. Each update is tagged with a lane (Sync, InputContinuous, Default, Transition, Idle...), and the scheduler picks the highest-priority lane with pending work.

The **scheduler** package (`scheduler`) time-slices: it works in 5ms-ish chunks and yields to the browser via `MessageChannel` so input and paint stay responsive. If higher-priority work comes in mid-render, the in-progress render is thrown away and restarted at the new priority.

This is what makes `useTransition` and `useDeferredValue` possible — they tag updates with the **Transition** lane so urgent input updates can interrupt them.

### Automatic batching

Since React 18, *all* updates inside the same tick are batched — including those from `setTimeout`, promises, and native event handlers. (Pre-18, only React event handlers batched.)

```tsx
function onClick() {
  setA(1);          // queued
  setB(2);          // queued
  fetch("/x").then(() => {
    setC(3);        // ALSO batched (18+)
    setD(4);
  });
}
```

To opt *out* (rare): `flushSync(() => setX(v))`.

### Hooks are a linked list

Each fiber stores a linked list of hook records — one per hook call, in call order. That's why hooks must be called in the same order every render: React doesn't track them by name, only by position.

```
fiber.memoizedState
  → hook1 (useState count)
    → hook2 (useState name)
      → hook3 (useEffect)
        → hook4 (useMemo total)
```

The "dispatcher" — the actual implementations of `useState`, `useEffect`, etc. — is swapped before each render based on context (mount vs update, RSC vs client). Calling a hook outside render hits the no-op dispatcher and throws.

### Commit phase — three sub-phases

1. **Before-mutation** — `getSnapshotBeforeUpdate` (class), prepare for DOM mutations.
2. **Mutation** — apply DOM changes, run cleanup of previous `useLayoutEffect`s.
3. **Layout** — run new `useLayoutEffect`s, refs are attached *here*, browser hasn't painted yet.
4. **Passive (async)** — `useEffect`s run after paint, on the next idle tick.

This is why `useLayoutEffect` blocks paint and `useEffect` doesn't.

### Concurrent rendering

The umbrella term for: time-slicing, interruption, transitions, Suspense-driven boundaries, and selective hydration. Concurrent features are opt-in per-update — code that never calls `startTransition` runs synchronously, exactly like older React.

### React Compiler — what it actually does

The compiler reads each component, builds an intermediate representation, and figures out which values can be cached between renders. It then emits the equivalent of fine-grained `useMemo`/`useCallback` — *but* at expression granularity, not just at the top level. Net effect: rerenders skip work the developer would have had to memoize by hand. You write the simple version; the compiler ships the fast version.

### Server Components, briefly

RSCs render on the server (or at build time), serialize a tree-shaped wire format, and stream it to the client. Client components hydrate normally; server components never ship JS. The boundary is a `"use client"` directive at the top of the file.

---

## Anti-patterns

- **Index-as-key on dynamic lists.** Breaks on reorder, insert, or remove — silently corrupts state.
- **Effects that mirror props into state.** Almost always means you wanted a derived value.
- **Mutating state objects.** `state.items.push(x)` doesn't re-render; React compares by reference.
- **Fetching in `useEffect` for everything.** Use the framework's data layer (RSC, route loaders, or a query lib) — `useEffect` fetching causes waterfalls and races.
- **Conditional hooks.** `if (cond) useState(...)` — breaks the linked list. Only `use()` is allowed conditionally.
- **`useMemo`/`useCallback` everywhere "for performance."** Each one has a cost. With the compiler, most are dead weight.
- **One giant context.** Every consumer re-renders on any change. Split by update frequency.
- **`useState` for derived data.** `const [doubled, setDoubled] = useState(x * 2)` — just write `const doubled = x * 2`.
- **`forwardRef` in new code.** Accept `ref` as a prop in 19+.
- **Treating Strict Mode double-renders as a bug.** They're how React surfaces missing cleanup or impure render logic.
- **Fighting the reconciler with manual DOM ops.** If you find yourself reaching for `document.querySelector` inside a component, lift it into a ref or — better — re-express the problem in terms of state.
- **Putting business logic in `useEffect` cleanup.** Cleanup runs in unexpected places (Strict Mode, transitions). Keep it idempotent.
- **Reading context in tight loops.** Every consumer re-renders on change; selector libs (`use-context-selector`, Zustand) exist for a reason.

---

## Checklist

- [ ] Strict Mode is enabled in dev.
- [ ] Lists use stable, unique `key`s — never index unless the list truly is static.
- [ ] No `useEffect` mirrors props into state.
- [ ] Async work in event handlers or transitions, not in `useEffect`, where possible.
- [ ] Context value is memoized; large contexts are split by change frequency.
- [ ] Components default to *not* memoized; `memo`/`useMemo` only added with a measured reason (or auto-applied by the compiler).
- [ ] Suspense boundaries wrap each async leaf with a meaningful fallback.
- [ ] Error boundaries wrap each routable section.
- [ ] Forms use Actions + `useActionState` + `useFormStatus` (or your framework's equivalent).
- [ ] `ref` is accepted as a prop, not via `forwardRef`, in new components.
- [ ] No conditional hook calls except `use()`.
- [ ] DevTools Profiler has been opened at least once on the hot screens.
- [ ] React Compiler enabled (or a conscious decision made not to).

---

## Further reading

- [React Docs (react.dev)](https://react.dev) — the reference, finally good.
- [React 19 Release Notes](https://react.dev/blog/2024/12/05/react-19) — Actions, `use`, ref-as-prop, metadata, the compiler.
- [React Compiler docs](https://react.dev/learn/react-compiler).
- [Andrew Clark — Concurrent React explained](https://github.com/reactwg/react-18/discussions/27).
- [Dan Abramov — A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/) — older but still the best mental model.
- [Lin Clark — A Cartoon Intro to Fiber](https://www.youtube.com/watch?v=ZCuYPiUIONs) — dated but the visuals stuck.
- [Mark Erikson — A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/).
- [The Rules of React](https://react.dev/reference/rules) — purity, idempotence, ordering. Worth re-reading every six months.
- [React internals deep dive — Bytedance team's articles](https://github.com/bvaughn/react-resources).

---

## Closing thought

React 19 is the version where the framework finally gets out of your way: Actions absorb most form boilerplate, `use` ends the data-fetching wrapper era, and the compiler removes the memoization tax. The under-the-hood story — fibers, lanes, double-buffered trees, a scheduler that yields to the browser — is what makes any of that possible, but you almost never have to think about it.

What I tell people new to React 19: write the simple, obvious code. Derive instead of sync. Trust the compiler. Reach for `useEffect` last, not first. The framework has spent a decade learning what to do for you — let it.
