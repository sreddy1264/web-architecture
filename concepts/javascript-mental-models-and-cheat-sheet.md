# JavaScript: Mental Models & Cheat Sheet

> A working architect's reference — the syntax I reach for daily, the mental models that explain why JS behaves the way it does, and the DOM/browser APIs that actually show up in production.

JavaScript is the language we ship to every device on Earth. Most bugs aren't in syntax — they're in mental models: a misunderstood `this`, a forgotten microtask, a closure capturing the wrong variable, a `==` quietly coercing. This guide is the cheat sheet plus the *why* underneath, so the surface stops surprising you.

---

## Why this matters

JavaScript looks like every other C-family language until it doesn't. Hoisting moves declarations. `this` is decided at call time, not definition. Equality coerces. Iteration order depends on key type. Promises resolve asynchronously even when they're already resolved. Memory leaks hide behind closures.

In practice: every JS senior I know has the same library of mental models — execution context, prototype chain, event loop, closures, coercion. The syntax is the easy part; those models are what separate "I can write JS" from "I can debug JS."

---

## Mindset

- **Read JavaScript bottom-up.** Engines hoist declarations, queue microtasks, and resolve `this` at call time. Code doesn't run in source order.
- **Default to `const`, fall back to `let`, never `var`.** Block scope is what you want; `var` is a footgun preserved for backwards compatibility.
- **Triple-equals always.** Never use `==` unless you've consciously decided you want coercion (you almost never do).
- **Mutability is opt-in.** Treat objects and arrays as immutable by default; clone with spread or `structuredClone` before changing.
- **Async is contagious.** Once a function is `async`, every caller becomes async-shaped. Decide where the async boundary lives.
- **The DOM is slow because layout is global.** Read once, batch writes, prefer transforms over geometry changes.
- **Prefer the platform.** `URL`, `FormData`, `AbortController`, `IntersectionObserver`, `structuredClone` — the platform ships these so you don't have to.

---

## The cheat sheet

### Variables & types

```js
const x = 1;        // block-scoped, no rebinding (the binding, not the value)
let y = 2;          // block-scoped, rebindable
var z = 3;          // function-scoped, hoisted — avoid

// Primitives (passed by value, immutable):
//   string, number, bigint, boolean, undefined, null, symbol
// Reference (passed by reference):
//   object (incl. arrays, functions, dates, regex, Map, Set, ...)

typeof null;        // "object"  ← legacy bug, can't be fixed
typeof [];          // "object"  ← use Array.isArray()
typeof NaN;         // "number"
Number.isNaN(NaN);  // true   ← prefer over global isNaN()
```

### Operators worth memorizing

```js
a ?? b              // nullish coalescing — b only if a is null/undefined
a?.b?.c             // optional chaining — short-circuits on null/undefined
a?.()               // optional call
a?.[k]              // optional index
a ||= b             // logical assignment: a = a || b
a &&= b             // a = a && b
a ??= b             // a = a ?? b
...spread           // expand array/object/iterable
{ ...rest }         // collect remaining keys
```

### Control flow

```js
// for...of iterates values; for...in iterates keys (don't use on arrays).
for (const item of items) { ... }
for (const key in obj) { ... }

// switch with strict matching:
switch (kind) {
  case "a": return 1;
  case "b": return 2;
  default:  return 0;
}

// try/catch/finally — `finally` runs even on return.
try { ... } catch (e) { ... } finally { ... }

// Labeled break for nested loops (rare but useful):
outer: for (...) for (...) if (cond) break outer;
```

### Functions

```js
function named() {}                        // hoisted
const fn = function () {};                 // not hoisted
const arrow = () => {};                    // no `this`, no `arguments`, can't `new`
const single = x => x * 2;                 // single-param parens optional
const obj = { method() {} };               // shorthand
const factory = (a, b = 1, ...rest) => ... // default + rest

// IIFE — immediately-invoked.
(() => { ... })();
```

A caveat: arrow functions inherit `this` from the enclosing lexical scope. They are not a syntactic shorthand — they have semantically different binding.

### Objects

```js
const user = {
  id: 1,
  name: "Ada",
  greet() { return `hi, ${this.name}`; },   // method shorthand
  [computed]: value,                         // computed key
};

// Copy — shallow.
const copy = { ...user };
const copy2 = Object.assign({}, user);

// Deep clone (structured-cloneable values only — no functions, no DOM).
const deep = structuredClone(user);

// Iterate.
Object.keys(user);   // own enumerable string keys
Object.values(user);
Object.entries(user);
Object.fromEntries([["a", 1], ["b", 2]]);

// Freeze / seal.
Object.freeze(user);   // shallow — nested objects still mutable
```

### Arrays

```js
const xs = [1, 2, 3];

xs.map(x => x * 2);               // [2, 4, 6]
xs.filter(x => x > 1);
xs.find(x => x > 1);              // 2
xs.findIndex(x => x > 1);         // 1
xs.some(x => x > 5);              // false
xs.every(x => x > 0);             // true
xs.reduce((acc, x) => acc + x, 0);
xs.flat();
xs.flatMap(x => [x, x]);
xs.includes(2);

// Mutating — generally avoid:
xs.push(4); xs.pop();
xs.unshift(0); xs.shift();
xs.sort(); xs.reverse(); xs.splice(1, 1);

// Non-mutating equivalents (2023+):
xs.toSorted();
xs.toReversed();
xs.toSpliced(1, 1);
xs.with(0, 99);                   // copy + replace at index

// Build an array.
Array.from({ length: 5 }, (_, i) => i);   // [0,1,2,3,4]
Array.of(1, 2, 3);
```

### Maps, Sets, Weak collections

```js
const m = new Map();
m.set(key, value);                // any key type, including objects
m.get(key); m.has(key); m.delete(key);
for (const [k, v] of m) { ... }   // insertion order

const s = new Set([1, 2, 3]);
s.add(4); s.has(2); s.delete(1);

// Deduplicate an array.
[...new Set(arr)];

// WeakMap / WeakSet — keys held weakly; great for attaching metadata
// to objects without preventing GC. Not iterable.
const meta = new WeakMap();
meta.set(domNode, { lastClick: Date.now() });
```

A reframe: reach for `Map` over plain object whenever keys are user-controlled, dynamic, or non-string. `Map` has no prototype-pollution risk and preserves insertion order across all key types.

### Strings & template literals

```js
const name = "Ada";
const s = `Hello, ${name}!`;                       // interpolation
const multi = `line 1
line 2`;                                            // multi-line

// Tagged templates — function call with parts.
function html(strings, ...values) { ... }
const safe = html`<div>${userInput}</div>`;

// Useful methods.
"abc".at(-1);                  // "c"  ← negative indexing
"  x  ".trim();
"a,b".split(",");
"abc".replaceAll("a", "x");
"abc".padStart(5, "0");        // "00abc"
"abc".repeat(3);
```

### Destructuring, spread, rest

```js
const { a, b: renamed, c = 1, ...rest } = obj;
const [first, second, ...tail] = arr;

// Function params.
function update({ id, ...patch }) { ... }

// Swap.
[a, b] = [b, a];

// Merge.
const merged = { ...defaults, ...overrides };
```

### Modules (ESM)

```js
// named
export const x = 1;
export function fn() {}
import { x, fn } from "./mod.js";
import { x as xx } from "./mod.js";

// default
export default class {}
import Whatever from "./mod.js";

// re-export
export * from "./mod.js";
export { x } from "./mod.js";

// dynamic
const mod = await import("./heavy.js");

// import attributes (2024+)
import data from "./data.json" with { type: "json" };
```

### Promises & async/await

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => resolve(value), 100);
});

p.then(v => ...).catch(e => ...).finally(() => ...);

// async/await — sequential by default.
async function load() {
  try {
    const a = await fetchA();
    const b = await fetchB(a);
    return b;
  } catch (e) { ... }
}

// Run in parallel — kick off both, then await.
const [a, b] = await Promise.all([fetchA(), fetchB()]);

// Combinators.
Promise.all([...])         // all succeed, or first reject
Promise.allSettled([...])  // wait for all, never rejects
Promise.race([...])        // first to settle (resolve or reject)
Promise.any([...])         // first to resolve, AggregateError if all reject

// Imperative resolve from outside (2024+).
const { promise, resolve, reject } = Promise.withResolvers();
```

### Iterators & generators

```js
// Iterable: anything with [Symbol.iterator].
for (const x of iterable) { ... }

// Generator — function* yields lazy values.
function* range(n) {
  for (let i = 0; i < n; i++) yield i;
}
[...range(3)];   // [0, 1, 2]

// Async iterator — for await...of.
async function* lines(stream) {
  for await (const chunk of stream) yield chunk.toString();
}
for await (const line of lines(stream)) { ... }
```

### Errors

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

// Cause chain (ES2022).
throw new Error("upload failed", { cause: originalError });

// AggregateError — multiple errors at once (Promise.any, custom).
throw new AggregateError([e1, e2], "multiple failed");
```

A caveat: throwing non-Error values (`throw "oops"`) loses stack traces and breaks tooling. Always throw `Error` instances.

### Modern (2023–2025) — quick reference

| Feature | What it does |
|---|---|
| `Object.groupBy(arr, fn)` | Group array items into an object by key |
| `Map.groupBy(arr, fn)` | Same, but returns a `Map` (object keys allowed) |
| `Array#toSorted/toReversed/toSpliced/with` | Non-mutating array versions |
| `Promise.withResolvers()` | Imperative `resolve`/`reject` outside the executor |
| `structuredClone(value)` | Deep clone — handles `Map`, `Set`, `Date`, typed arrays |
| `Error({ cause })` | Wrap and chain errors |
| `String#at(i)` / `Array#at(i)` | Negative-index access |
| `import ... with { type: "json" }` | Import attributes |
| Top-level `await` | At ESM module top level only |
| `#private` fields, `static {}` blocks | Real privacy in classes |
| `RegExp` `/v` flag | Set notation, intersection, difference |
| `Iterator.prototype` helpers (Stage 4) | `.map`, `.filter`, `.take` directly on iterators |
| Decorators (Stage 3) | `@decorator` on classes, methods, fields |

---

## Mental models

### Execution context, scope chain, hoisting, TDZ

When a function runs, the engine creates an **execution context** with:

- A **lexical environment** (the local variables + a reference to the parent scope).
- A `this` binding.
- A reference to outer scopes — the **scope chain**.

Before any code runs, the engine *hoists* declarations:

```js
console.log(a); // undefined — `var a` is hoisted, value isn't
var a = 1;

console.log(b); // ReferenceError — TDZ
let b = 1;

foo();          // works — function declarations hoist fully
function foo() {}
```

- `var` → hoisted, initialized to `undefined`.
- `let` / `const` → hoisted but **uninitialized**. Reading them before declaration is a **Temporal Dead Zone** error.
- `function` declaration → hoisted with body.
- Class declaration → hoisted in TDZ.

### Closures

A closure is a function bundled with the lexical environment it was created in. The function keeps that environment alive for as long as the function is reachable.

```js
function counter() {
  let n = 0;
  return () => ++n;
}
const c = counter();
c(); c(); c();   // 3
```

In practice: closures are how every callback, event handler, and module-level private state works. They're also the most common source of memory leaks — long-lived closures keep their entire scope alive.

### `this` binding

`this` is determined at *call time*, by **how** the function is called:

| Call shape | `this` is |
|---|---|
| `obj.method()` | `obj` |
| `fn()` | `undefined` (strict) / global (sloppy) |
| `new Fn()` | the new instance |
| `fn.call(ctx)` / `fn.apply(ctx)` / `fn.bind(ctx)` | `ctx` |
| Arrow function | inherited from enclosing lexical scope |
| DOM event handler (non-arrow) | the element |

```js
const user = {
  name: "Ada",
  greet() { return this.name; }
};

const fn = user.greet;
fn();                  // undefined — `this` is detached
user.greet();          // "Ada"
fn.call(user);         // "Ada"

// Arrows can't be detached.
const obj = {
  name: "Ada",
  greet: () => this?.name   // `this` is the enclosing scope, NOT obj
};
```

### Prototype chain & `class`

Every object has a hidden `[[Prototype]]` — accessible via `Object.getPrototypeOf(obj)` (or `__proto__`, deprecated). Property lookups walk the chain until they find a match or hit `null`.

```js
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}
class Dog extends Animal {
  speak() { return `${this.name} barks`; }
}

// Equivalent prototype chain:
//   dog → Dog.prototype → Animal.prototype → Object.prototype → null
```

Worth knowing: `class` is syntactic sugar over prototype-based inheritance. Methods live on `Prototype`, instance properties live on the instance. Private fields (`#x`) are the first non-sugar addition — true privacy enforced by the engine.

### Reference vs value

```js
let a = { n: 1 };
let b = a;                  // same reference
b.n = 2;
a.n;                        // 2

let x = 1;
let y = x;                  // copy
y = 2;
x;                          // 1

// Function args follow the same rules — primitives copy, objects share.
function reset(o) { o.n = 0; }
reset(a);                   // mutates the caller's object
```

A reframe: "passed by reference" in JS really means "the reference itself is passed by value." Reassigning the parameter doesn't affect the caller; mutating its contents does.

### Type coercion

```js
// == coerces. === doesn't.
0 == "0";       // true
0 == [];        // true
"" == false;    // true
null == undefined;   // true
null == 0;      // false  ← null only equals undefined under ==

// Falsy values (everything else is truthy):
//   false, 0, -0, 0n, "", null, undefined, NaN

// Conversion explicitly:
Number("12");   // 12
Number("");     // 0  ← surprise
Number(" ");    // 0  ← surprise
Number(null);   // 0
Number(undefined); // NaN
String(null);   // "null"
Boolean("0");   // true   ← non-empty string
```

Lesson learned: `==` has at least 100 special cases. Just don't use it. The `eqeqeq` lint rule is non-negotiable.

### The event loop, microtasks vs macrotasks

JS runs single-threaded. The event loop processes one task at a time. After each task, **all** queued microtasks drain before the next task or render.

```
┌──────────────────┐
│  Macrotask queue │  setTimeout, setInterval, I/O, UI events
└────────┬─────────┘
         │  pick one
         ▼
┌──────────────────┐
│   Run task       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Drain microtasks │  Promise.then, queueMicrotask, MutationObserver
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Render (maybe)  │  rAF callbacks, style/layout/paint
└──────────────────┘
```

```js
console.log("a");
setTimeout(() => console.log("b"), 0);
Promise.resolve().then(() => console.log("c"));
console.log("d");
// → a, d, c, b
```

Worth knowing: a `Promise.then` chain that never yields will starve rendering. If you find yourself thinking "why isn't the spinner appearing?" — it's probably this.

### Promises under the hood

A promise has three states: `pending`, `fulfilled`, `rejected`. Once settled, it's immutable.

- `.then(onFulfilled, onRejected)` always returns a **new** promise.
- Throwing inside `.then` rejects the returned promise.
- Returning a promise from `.then` makes the chain wait for it (no double wrapping — promises don't nest).
- `.catch(fn)` is sugar for `.then(undefined, fn)`.
- `.finally(fn)` runs on both success and failure, doesn't change the value.

A caveat: an unhandled rejection is a runtime error in modern environments. Always attach a `.catch` (or wrap `await` in `try/catch`).

### `async/await` desugaring

`async function f()` is roughly:

```js
async function f() {
  const a = await x;
  return a + 1;
}

// ≈

function f() {
  return Promise.resolve(x).then(a => a + 1);
}
```

Each `await` yields back to the event loop and resumes when the promise settles. Errors thrown after `await` are caught by surrounding `try/catch` because they're translated into `.catch` on the underlying promise.

### Modules (ESM vs CommonJS)

| | ESM | CommonJS |
|---|---|---|
| Syntax | `import` / `export` | `require` / `module.exports` |
| Resolution | Static (analyzable at parse) | Dynamic (runtime) |
| Async | Yes (top-level await) | No |
| Live bindings | Yes — imports update if exporter reassigns | No — copies |
| Tree-shakable | Yes | No (mostly) |
| `this` at top level | `undefined` | the module exports |

In practice: ESM is the default everywhere new (browser, Bun, Deno, modern Node). CJS is the legacy substrate of npm — interop is fine but mixed-mode (especially default exports) still has sharp edges.

### Memory model & GC

V8 (and most engines) use generational, mark-and-sweep garbage collection. You can't free memory manually; you can only stop *referencing* it.

Common leaks:

- Long-lived event listeners attached to short-lived components (use `AbortController` to remove).
- Closures capturing big objects accidentally (`{ data, helper: () => ... }` keeps `data` alive even if only `helper` is held).
- Forgotten `setInterval`/`setTimeout`.
- Detached DOM nodes still referenced from JS state.
- Caches that grow unbounded (use `WeakMap` / `WeakRef` if appropriate).

```js
// AbortController — cancel anything that takes a signal.
const ctrl = new AbortController();
el.addEventListener("click", onClick, { signal: ctrl.signal });
fetch(url, { signal: ctrl.signal });
// later:
ctrl.abort();
```

---

## DOM cheat sheet

### Selecting & creating

```js
document.querySelector(".btn");             // first match
document.querySelectorAll(".item");         // static NodeList
document.getElementById("root");            // fastest single lookup

const el = document.createElement("div");
el.className = "card";
el.dataset.id = "42";                       // → data-id="42"
el.textContent = "hi";                      // safe (no parsing)
el.innerHTML = "<b>hi</b>";                 // parses HTML — XSS risk if input is untrusted
```

A caveat: never use `innerHTML` with user-controlled strings. Use `textContent`, or sanitize via DOMPurify, or use `setHTML()` (Sanitizer API where available).

### Mutating

```js
parent.append(child);                       // append — accepts strings + nodes
parent.prepend(child);
ref.before(node);
ref.after(node);
old.replaceWith(neu);
node.remove();

el.classList.add("active");
el.classList.toggle("open", isOpen);
el.classList.contains("active");

el.setAttribute("aria-expanded", "true");
el.removeAttribute("hidden");
```

### Events

```js
// Modern: AbortController for clean removal of multiple listeners.
const ctrl = new AbortController();
el.addEventListener("click", handler, { signal: ctrl.signal });
window.addEventListener("scroll", onScroll, { signal: ctrl.signal, passive: true });
ctrl.abort();   // removes all at once

// Once.
el.addEventListener("click", handler, { once: true });

// Capture phase.
el.addEventListener("click", handler, { capture: true });

// Delegation — single listener on the parent.
list.addEventListener("click", e => {
  const item = e.target.closest("[data-id]");
  if (!item) return;
  doStuff(item.dataset.id);
});

// Custom events.
el.dispatchEvent(new CustomEvent("ready", { detail: { ok: true }, bubbles: true }));
```

Worth knowing: `passive: true` on `scroll`/`touchmove` tells the browser the listener won't `preventDefault`, so it can scroll without waiting for JS — huge for scroll perf.

### Forms

```js
form.addEventListener("submit", e => {
  e.preventDefault();
  const data = Object.fromEntries(new FormData(form));
  // → { name: "Ada", role: "engineer" }
});

input.addEventListener("input", e => e.target.value);
input.addEventListener("change", ...);     // fires on commit (blur for text)

// Validation API.
input.checkValidity();
input.setCustomValidity("Pick a stronger password");
form.reportValidity();
```

### Observers

```js
// Visible? Lazy-load images, infinite scroll, animations on enter.
const io = new IntersectionObserver(entries => {
  for (const e of entries) {
    if (e.isIntersecting) load(e.target);
  }
}, { rootMargin: "200px" });
io.observe(el);

// Watch DOM changes.
const mo = new MutationObserver(records => { ... });
mo.observe(el, { childList: true, subtree: true, attributes: true });

// Watch element size — replaces window.resize for component-level sizing.
const ro = new ResizeObserver(entries => { ... });
ro.observe(el);

// Performance entries (LCP, INP, long tasks).
new PerformanceObserver(list => list.getEntries().forEach(...)).observe({ type: "largest-contentful-paint", buffered: true });
```

### Measuring (and not thrashing)

```js
const rect = el.getBoundingClientRect();    // top, left, width, height (subpixel, post-transform)
el.offsetWidth; el.offsetHeight;            // integer, includes border
el.clientWidth; el.clientHeight;            // integer, excludes border
el.scrollTop; el.scrollHeight;
```

A caveat — **layout thrashing**: reads (`offsetWidth`, `getBoundingClientRect`) force the browser to flush pending style/layout. Interleaving reads and writes in a loop is O(n²) layout work. Read all measurements first, then do all writes.

### Templates, `<dialog>`, Shadow DOM

```html
<template id="row">
  <li class="row"><span></span></li>
</template>
```

```js
const tpl = document.getElementById("row");
const node = tpl.content.cloneNode(true);
node.querySelector("span").textContent = name;
list.append(node);

// Native modal — focus-trapped, ESC-dismissable.
dialog.showModal();
dialog.close();

// Shadow DOM — encapsulated styles + structure.
const root = el.attachShadow({ mode: "open" });
root.innerHTML = `<style>:host { ... }</style><slot></slot>`;
```

### Scheduling

```js
queueMicrotask(() => ...);                  // run after current task, before next
requestAnimationFrame(t => ...);            // before next paint, t is DOMHighResTimeStamp
requestIdleCallback(deadline => ...);       // when browser is idle (Chromium)
setTimeout(fn, 0);                          // next macrotask — slower than microtask
scheduler?.postTask(fn, { priority: "user-blocking" }); // newer, priority-aware
```

### Web APIs worth knowing by heart

```js
// fetch + AbortController.
const res = await fetch(url, { signal });
if (!res.ok) throw new Error(res.statusText);
const data = await res.json();

// URL / URLSearchParams.
const u = new URL("/api/users", location.origin);
u.searchParams.set("page", "2");
fetch(u);

// FormData (also works with multipart).
const fd = new FormData();
fd.append("file", fileInput.files[0]);
await fetch("/upload", { method: "POST", body: fd });

// structuredClone — true deep clone.
const copy = structuredClone(state);

// Cross-context messaging.
const ch = new BroadcastChannel("auth");
ch.postMessage({ type: "logout" });
ch.onmessage = e => { ... };

// Web Storage.
localStorage.setItem("k", JSON.stringify(v));
sessionStorage.getItem("k");
```

---

## Anti-patterns

- **Using `==` "because it's shorter."** It's never shorter than the bug it eventually causes.
- **`var` in new code.** Block scope is what you want; `const`/`let` make refactoring safer.
- **`for...in` on arrays.** Iterates inherited keys + indices as strings. Use `for...of` or `.forEach`.
- **Forgetting `await` inside `.map`.** `arr.map(async ...)` returns an array of promises; `Promise.all` it or use a `for...of`.
- **`async` function that returns nothing useful.** If you don't `return`, callers get an undefined-resolved promise; usually you wanted `await` instead.
- **Mutating function arguments.** Hard to debug; almost always a sign you wanted a return value.
- **`new Date(string)`.** Parsing is implementation-defined for non-ISO strings. Use `Date.parse(isoString)` or a date library.
- **Floating-point money.** `0.1 + 0.2 !== 0.3`. Use integers (cents) or `BigInt` or a decimal lib.
- **`innerHTML` with user input.** XSS waiting to happen. `textContent` or sanitize.
- **`setTimeout(fn, 0)` for microtasks.** Use `queueMicrotask` — it's deterministic and doesn't add a 4ms minimum.
- **Layout thrashing in loops.** Read all measurements, then write all DOM changes.
- **Listeners without removal.** Memory leaks on SPAs. Use `AbortController` or store a reference and call `removeEventListener`.
- **Catching everything and logging it away.** Errors you can't handle should propagate; errors you do handle should be specific.
- **Polling instead of observing.** `IntersectionObserver`/`ResizeObserver`/`MutationObserver` exist for reasons.

---

## Checklist

- [ ] Strict mode enabled (default in modules and classes).
- [ ] No `var`, no `==` in the codebase (lint rules enforce both).
- [ ] All async functions either `await` or explicitly return promises — no fire-and-forget.
- [ ] Promise chains have a `.catch` or live inside `try/catch`.
- [ ] DOM listeners attached with `AbortController` for clean removal.
- [ ] No `innerHTML` with untrusted strings.
- [ ] Long lists / images are virtualized or lazy-loaded with `IntersectionObserver`.
- [ ] Reads and writes are batched — no measure-mutate-measure loops.
- [ ] Heavy data structures use `Map`/`Set`/`WeakMap` over plain objects when keys are dynamic.
- [ ] Deep clones use `structuredClone`, not `JSON.parse(JSON.stringify(...))`.
- [ ] Errors thrown are `Error` instances, with `cause` when wrapping.
- [ ] No reliance on iteration order of plain objects when keys are user-controlled.
- [ ] Bundle ships ESM; tree-shaking is verified.

---

## Further reading

- [MDN Web Docs](https://developer.mozilla.org) — the reference. Keep a tab open.
- [TC39 Proposals](https://github.com/tc39/proposals) — what's coming, with stage tracking.
- [Jake Archibald — In the Loop](https://www.youtube.com/watch?v=cCOL7MC4Pl0) — the clearest event-loop talk on the internet.
- [Lin Clark — A Cartoon Intro to the Event Loop / V8](https://hacks.mozilla.org/2017/02/a-cartoon-intro-to-arraybuffers-and-sharedarraybuffers/).
- [Kyle Simpson — You Don't Know JS Yet](https://github.com/getify/You-Dont-Know-JS) — opinionated, deep, free.
- [V8 blog](https://v8.dev/blog) — the engine's perspective on what fast JS looks like.
- [Web.dev — Modern JavaScript](https://web.dev/articles/modern-javascript) — best-practices for shipping.
- [HTML Living Standard — DOM](https://dom.spec.whatwg.org/) — the spec, when MDN isn't enough.

---

## Closing thought

JavaScript rewards depth in a few specific places: closures, `this`, the event loop, the prototype chain, coercion. Get those right and the rest of the language stops surprising you. The DOM rewards a different muscle: knowing when to read, when to write, and when to let an observer do the watching for you.

What I tell people: master the mental models once, then keep the cheat sheet open. Don't memorize syntax — memorize *why* things behave the way they do, and the syntax stops mattering.
