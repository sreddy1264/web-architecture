# TypeScript Deep Dive

*Last reviewed: 2026-05*

> A practical guide to using TypeScript well — the type system, configuration, advanced patterns, React interop, performance, and the mindset that turns it from a tax into a tool.

---

## Table of Contents

1. [Why TypeScript Matters](#why-typescript-matters)
2. [The TypeScript Mindset](#the-typescript-mindset)
3. [`tsconfig.json` — Settings That Matter](#tsconfigjson--settings-that-matter)
4. [Foundations: Types vs. Values, Structural Typing, Inference](#foundations-types-vs-values-structural-typing-inference)
5. [`type` vs `interface`](#type-vs-interface)
6. [Literal Types, `const` Assertions, and `satisfies`](#literal-types-const-assertions-and-satisfies)
7. [Union & Intersection Types](#union--intersection-types)
8. [`any` vs `unknown` vs `never`](#any-vs-unknown-vs-never)
9. [Type Narrowing](#type-narrowing)
10. [Discriminated Unions](#discriminated-unions)
11. [Generics](#generics)
12. [Conditional Types & `infer`](#conditional-types--infer)
13. [Mapped Types](#mapped-types)
14. [Template Literal Types](#template-literal-types)
15. [Utility Types Reference](#utility-types-reference)
16. [Branded / Nominal Types](#branded--nominal-types)
17. [Variance & Function Types](#variance--function-types)
18. [Type-Only Imports & Module Resolution](#type-only-imports--module-resolution)
19. [Module Augmentation & Declaration Merging](#module-augmentation--declaration-merging)
20. [React with TypeScript](#react-with-typescript)
21. [Runtime Validation & The Type/Value Boundary](#runtime-validation--the-typevalue-boundary)
22. [Type-Level Programming](#type-level-programming)
23. [TypeScript Performance](#typescript-performance)
24. [Migration Strategies (JS → TS)](#migration-strategies-js--ts)
25. [Tooling Landscape](#tooling-landscape)
26. [Common Anti-Patterns](#common-anti-patterns)
27. [Quick Reference Checklist](#quick-reference-checklist)
28. [Further Reading](#further-reading)

---

## Why TypeScript Matters

### 1. Types catch the bugs you'd write today
Microsoft's own studies have shown that ~15% of bugs in JavaScript codebases are bugs TypeScript would have caught at compile time — typos, wrong arguments, accessing properties on undefined, off-by-one signatures. That's *one in seven* bugs that never reach a code review, never become an incident, never get filed.

### 2. Types are documentation that can't rot
A function signature is the only documentation that the compiler enforces. JSDoc decays; READMEs lie; types are mechanically truthful. Every refactor either updates the types or breaks the build.

### 3. Refactor confidence at scale
Renaming a property across a 200K-line codebase is a five-second operation in TypeScript and a multi-day audit in JavaScript. **The single biggest day-to-day quality-of-life win.**

### 4. Ergonomic API contracts
End-to-end-typed stacks (tRPC, GraphQL Codegen, OpenAPI generators) turn the API boundary into a refactor-safe interface. Backend renames a field, the frontend stops compiling.

### 5. Editor superpowers
Autocomplete, jump-to-definition, hover-to-inspect, in-place rename, automated imports — none of which work meaningfully without types. The IDE becomes a force multiplier.

### 6. AI agents read types better than humans do
LLMs producing code are dramatically more accurate against typed APIs. As coding agents become standard, **types are part of how you communicate with the machines that write code on your behalf**, not just the humans.

---

## The TypeScript Mindset

### 1. Strict mode is non-negotiable
`strict: true` is the entry ticket. Without it, you have JavaScript with hints. With it, you have a tool that catches real bugs. Turn it on at project creation; bake it into the template.

### 2. Lean on inference; annotate on boundaries
TypeScript's inference is excellent. Annotate function *return types* (boundary contract) and *exported types* (consumer-facing). Let inference handle local variables.

```ts
// ❌ Annotation everywhere — noise
const total: number = items.reduce((sum: number, item: Item): number => sum + item.price, 0);

// ✅ Inference handles the body; signature explicit at the boundary
function sumPrices(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

### 3. Types describe behavior, not just shape
A `User` type with optional `email` describes a domain reality (email may be missing). A `LoggedInUser` type without optional `email` is a different domain reality. Type design is data modeling, not just shape annotation.

### 4. Don't fight the compiler — interrogate it
When TypeScript says "X is not assignable to Y," it's almost always right. Read the error carefully; the bug is usually in the code, not the type. The few times the compiler is wrong, narrow with a type guard or a typed cast — never with `any`.

### 5. `any` is a contagion
One `any` infects every callsite. The discipline that separates good codebases from bad ones is **count of explicit `any` references** — and the trajectory.

### 6. Types should be load-bearing, not decorative
If a type can be removed without breaking anything, it wasn't doing useful work. Aim for types that, if removed, would expose real bugs.

### 7. Runtime is the one place TypeScript can't help
Types disappear at runtime. Validate at the boundary (network, file, env, user input) with a runtime schema (Zod, Valibot). Inside the trust boundary, types are sufficient.

---

## `tsconfig.json` — Settings That Matter

The config file is where most TypeScript pain originates. Get it right once.

### The non-negotiables
```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",   // "NodeNext" if not using a bundler
    "strict": true,                  // turns on the full strict suite
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  }
}
```

### What each one buys you

| Flag | Catches |
|---|---|
| `strict` | Umbrella for `strictNullChecks`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitAny`, `noImplicitThis`, `alwaysStrict`, `useUnknownInCatchVariables` |
| `noUncheckedIndexedAccess` | `arr[i]` is `T \| undefined`, not `T`. Catches off-by-one and missing-key bugs that `strict` alone misses. **One of the highest-leverage flags.** |
| `exactOptionalPropertyTypes` | Distinguishes "property missing" from "property is undefined." Closes a subtle hole. |
| `noImplicitOverride` | `override` keyword required when overriding a parent method. Prevents accidental rename drift. |
| `verbatimModuleSyntax` | Forces explicit `import type` for type-only imports. Cleanest interop with bundlers and Node's ESM. |
| `isolatedModules` | Ensures every file can be transpiled in isolation. Required for esbuild, swc, Babel. |
| `skipLibCheck` | Skip type-checking `.d.ts` files in `node_modules`. Massive build-time win; the cost is missing the rare lib-to-lib type clash. |

### Strict additions to consider per project
- `noUnusedLocals`, `noUnusedParameters` — annoying during dev, valuable in CI. Many teams scope these to CI-only via a separate `tsconfig.ci.json`.
- `allowUnreachableCode: false`, `allowUnusedLabels: false`
- `noPropertyAccessFromIndexSignature` — `obj["key"]` vs `obj.key` distinction for index-signature types

### Project structure flags
- `paths` + `baseUrl` for module aliases (`@/components/...`)
- `references` for **TypeScript Project References** (monorepos with incremental builds)
- `incremental: true` + `tsBuildInfoFile` for incremental compilation

### A tip
Don't override `lib` unless you know what you're doing. The defaults track `target`. Manual overrides are a common cause of "DOM types missing" and "ES2023 features missing."

---

## Foundations: Types vs. Values, Structural Typing, Inference

### Two parallel languages
TypeScript has a **type-level language** and a **value-level language** that don't intersect. `5` is a value; `number` is a type; `5 as const` is a value whose *type* is the literal `5`. Holding this mental model fixes 80% of beginner confusion.

```ts
const x = 5;          // x: 5 (literal type, since const)
let y = 5;            // y: number (widened, since let is mutable)
type T = typeof x;    // T = 5 — `typeof` operates at the type level
```

### Structural typing
TypeScript types are about **shape**, not name. Two types with identical structure are interchangeable, regardless of declaration:

```ts
type Point2D = { x: number; y: number };
interface Coord { x: number; y: number }

const p: Point2D = { x: 1, y: 2 };
const c: Coord = p;  // ✅ assignable — same shape
```

This is why "duck typing" works at the type level. It's fast and ergonomic, but it leaks: see [Branded Types](#branded--nominal-types) for when you need to fight back.

### Inference flows from values to types
```ts
const config = { host: 'localhost', port: 3000 };
// config: { host: string; port: number }

const config2 = { host: 'localhost', port: 3000 } as const;
// config2: { readonly host: 'localhost'; readonly port: 3000 }
```

The `as const` assertion preserves literal types and freezes the structure. Use it for configuration tables, route maps, and discriminated-union constants.

---

## `type` vs `interface`

The most-asked TypeScript question, with a small set of real differences.

| | `type` | `interface` |
|---|---|---|
| Object shapes | ✅ | ✅ |
| Unions | ✅ | ❌ |
| Intersections | ✅ | ✅ (via `extends`) |
| Tuples | ✅ | ❌ (awkward) |
| Mapped types | ✅ | ❌ |
| Conditional types | ✅ | ❌ |
| Declaration merging | ❌ | ✅ |
| Performance (large unions) | sometimes slower | sometimes faster |

### The pragmatic rule
- **Use `interface`** for public-facing object shapes that consumers might extend (component props, API DTOs, library exports).
- **Use `type`** for everything else (unions, intersections, mapped types, tuple types, function types, complex composition).

If you're writing a library, prefer `interface` for object types — it lets consumers augment your types via [declaration merging](#module-augmentation--declaration-merging).

### One rule both follow
**Don't pick differently from one file to the next.** Pick a project default and lint for it.

---

## Literal Types, `const` Assertions, and `satisfies`

### Literal types
A type can be a *single value*:
```ts
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type Port = 80 | 443 | 8080;
```

This is how TypeScript replaces enums (which have runtime cost and weird semantics — see anti-patterns).

### `as const`
Freezes inference to literal types and makes properties `readonly`:
```ts
const colors = ['red', 'green', 'blue'] as const;
// colors: readonly ['red', 'green', 'blue']
type Color = typeof colors[number];   // 'red' | 'green' | 'blue'

const routes = {
  home: '/',
  profile: '/profile',
  settings: '/settings',
} as const;
type Route = typeof routes[keyof typeof routes];   // '/' | '/profile' | '/settings'
```

This pattern — `as const` + `typeof X[keyof typeof X]` — is the canonical way to derive a union from a JS object. Use it constantly.

### `satisfies`
TypeScript 4.9+ introduced `satisfies`, which validates that a value matches a type *without widening it*:

```ts
type Config = Record<string, string | number>;

// ❌ Without satisfies — port is widened to string|number
const config: Config = { host: 'localhost', port: 3000 };
config.port.toFixed(2);   // Error: toFixed not on string|number

// ✅ With satisfies — type is preserved as the inferred shape
const config2 = { host: 'localhost', port: 3000 } satisfies Config;
config2.port.toFixed(2);  // ✅ port is number
```

`satisfies` is the right tool for "I want this to conform to a shape, but I want the precise inferred type for usage." Use it for config objects, route tables, theme objects, and anywhere you want both type safety *and* literal precision.

---

## Union & Intersection Types

### Unions
"This value is one of these types":
```ts
type ID = string | number;
type Status = 'idle' | 'loading' | 'success' | 'error';
```

To use a union, you must **narrow** it (see next section).

### Intersections
"This value is *all* of these types simultaneously":
```ts
type WithTimestamps = { createdAt: Date; updatedAt: Date };
type User = { id: string; name: string };
type UserRecord = User & WithTimestamps;
// { id: string; name: string; createdAt: Date; updatedAt: Date }
```

Intersections compose mixins; unions express choices. Don't confuse them.

### Distributed unions
Conditional types and many built-ins distribute over unions. This is sometimes what you want, sometimes a footgun:

```ts
type ToArray<T> = T extends any ? T[] : never;
type R = ToArray<string | number>;   // string[] | number[] — distributed!

// To prevent distribution, wrap in tuple:
type ToArrayNoDist<T> = [T] extends [any] ? T[] : never;
type R2 = ToArrayNoDist<string | number>;   // (string | number)[]
```

---

## `any` vs `unknown` vs `never`

These three are the bottom of the type system. Knowing them well is the difference between using TypeScript and *using* TypeScript.

### `any` — the escape valve
Disables all checking. Assignable to and from anything:
```ts
const x: any = 'hello';
x.foo.bar.baz();   // No error. No safety. Bug guaranteed.
```

**Treat `any` as a code smell.** Every `any` is a hole in the type system. If you must use it, prefer `unknown`.

### `unknown` — the safe `any`
Holds any value, but you can't *do* anything with it without narrowing:
```ts
const x: unknown = JSON.parse(input);
// x.foo  ← error
if (typeof x === 'object' && x !== null && 'foo' in x) {
  // now narrowed to {foo: unknown}
}
```

`unknown` is what you want for the result of `JSON.parse`, `fetch().json()`, and other untrusted boundaries. Force callers to validate.

### `never` — the empty type
Represents "no value can ever exist":
```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never` is the bottom type — assignable to *every* type, but no value is assignable *to* it. The most useful pattern is **exhaustiveness checking**:

```ts
type Shape = { kind: 'circle'; r: number } | { kind: 'square'; side: number };

function area(s: Shape): number {
  switch (s.kind) {
    case 'circle': return Math.PI * s.r ** 2;
    case 'square': return s.side ** 2;
    default: {
      const _exhaustive: never = s;   // compile error if a new kind is added
      return _exhaustive;
    }
  }
}
```

If someone adds `{ kind: 'triangle' }` to `Shape` and forgets to handle it, the build fails. **This is one of the most useful patterns in TypeScript.**

---

## Type Narrowing

Given a union, you reduce it to a single member through control-flow analysis.

### `typeof` narrowing
```ts
function format(value: string | number): string {
  if (typeof value === 'string') return value.toUpperCase();
  return value.toFixed(2);
}
```

### `instanceof` narrowing
```ts
if (err instanceof Error) {
  console.error(err.message);
}
```

### `in` operator narrowing
```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

function speak(animal: Cat | Dog) {
  if ('meow' in animal) animal.meow();
  else animal.bark();
}
```

### Equality narrowing
```ts
function process(x: string | null) {
  if (x === null) return;
  // x is string here
}
```

### User-defined type guards (`is`)
For complex narrowing, write a function whose return type is `arg is T`:
```ts
function isUser(x: unknown): x is User {
  return typeof x === 'object'
      && x !== null
      && 'id' in x
      && 'email' in x;
}

if (isUser(payload)) {
  // payload is User here
}
```

⚠️ **Type guards lie if you let them.** The body of `isUser` could return `true` for any value; the compiler trusts your assertion. Pair complex guards with runtime schema validation (Zod) and let `parse()` narrow for you.

### Assertion functions (`asserts`)
A function that throws if the assertion fails, otherwise narrows:
```ts
function assertIsUser(x: unknown): asserts x is User {
  if (!isUser(x)) throw new Error('Not a user');
}

assertIsUser(payload);
// payload is User from here on
```

Use sparingly — they break early-return patterns and require explicit `asserts` annotation.

---

## Discriminated Unions

The single most important pattern in modern TypeScript. Tag each member with a literal field; narrow on the tag.

```ts
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function render(state: RequestState<User>) {
  switch (state.status) {
    case 'idle':    return <Idle />;
    case 'loading': return <Spinner />;
    case 'success': return <Profile user={state.data} />;   // narrowed!
    case 'error':   return <ErrorBox e={state.error} />;
  }
}
```

The compiler knows `state.data` exists *only* in the `success` branch and `state.error` exists *only* in the `error` branch. Every "boolean explosion" (`isLoading`, `isError`, `isSuccess`) should be reframed as a discriminated union.

> **In practice:** the rule of thumb is "if your state requires `if (x && !y && !z)`, you have a discriminated union waiting to be born."

---

## Generics

Type parameters that flow through functions, types, and classes.

### Generic functions
```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const n = first([1, 2, 3]);    // n: number | undefined
const s = first(['a', 'b']);   // s: string | undefined
```

### Generic types
```ts
type Result<T, E = Error> =
  | { ok: true;  value: T }
  | { ok: false; error: E };

function divide(a: number, b: number): Result<number> {
  if (b === 0) return { ok: false, error: new Error('div by zero') };
  return { ok: true, value: a / b };
}
```

Default type parameters (`E = Error`) make generics ergonomic. Use them.

### Constraints (`extends`)
Limit what types can be used:
```ts
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: 'Alice' };
getProp(user, 'name');    // ✅ string
getProp(user, 'unknown'); // ❌ Argument 'unknown' not assignable
```

`K extends keyof T` is one of the most useful constraint patterns.

### Generic ergonomics
- **Avoid generics that "look generic" but always resolve to one type.** Generics earn their keep when they let one function work for many input types.
- **Constrain everything.** `<T>` with no constraint usually means "I haven't thought about what T is."
- **Don't go overboard.** A function with five generic parameters is usually a sign the type is doing too much work. Refactor into smaller pieces.

---

## Conditional Types & `infer`

Conditional types are TypeScript's "if-else at the type level."

```ts
type IsString<T> = T extends string ? true : false;
type A = IsString<'hello'>;   // true
type B = IsString<42>;        // false
```

### `infer` extracts a piece
The `infer` keyword introduces a type variable inside a conditional, capturing part of the matched type:

```ts
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;
type R = ReturnTypeOf<() => string>;   // string

type ArrayElement<T> = T extends Array<infer U> ? U : never;
type E = ArrayElement<number[]>;       // number

// Pattern-match a string
type FirstWord<S> = S extends `${infer W} ${string}` ? W : S;
type W = FirstWord<'hello world'>;     // 'hello'
```

### Distributive conditional types (revisited)
When a conditional acts on a "naked" type parameter, it distributes over unions:
```ts
type NonNullable<T> = T extends null | undefined ? never : T;
type R = NonNullable<string | null>;   // string
```

That's how the built-in `NonNullable` works.

### When to reach for conditional types
- Library authors deriving types from generic inputs
- Mapped + conditional combinations (most utility types)
- Pattern-matching on string literals or function shapes

Most application code shouldn't write conditional types — utility types already cover the common cases.

---

## Mapped Types

Iterate over the keys of one type to produce another:

```ts
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

type Partial<T> = {
  [K in keyof T]?: T[K];
};

type Required<T> = {
  [K in keyof T]-?: T[K];   // -? removes the optional modifier
};
```

### Modifiers
- `readonly` / `-readonly` — add or remove
- `?` / `-?` — add or remove optionality

### Key remapping with `as` (TS 4.1+)
```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type Person = { name: string; age: number };
type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }
```

This unlocks a class of "rename-while-mapping" patterns that are essential for ergonomic library APIs.

---

## Template Literal Types

String types you can pattern-match and concatenate:

```ts
type Greeting = `Hello, ${string}!`;
const g1: Greeting = 'Hello, world!';   // ✅
const g2: Greeting = 'Hi';              // ❌

type EventName<T extends string> = `on${Capitalize<T>}`;
type ClickEvent = EventName<'click'>;   // 'onClick'
```

### Built-in string manipulators
`Uppercase<T>`, `Lowercase<T>`, `Capitalize<T>`, `Uncapitalize<T>`. Combine them with template literals for typed event handlers, route params, CSS-in-TS theming, and more.

### Pattern matching
```ts
type ExtractPath<S> = S extends `/${infer P}` ? P : never;
type R = ExtractPath<'/users/123'>;     // 'users/123'

type Params<S> =
  S extends `${infer Pre}/:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof Params<`/${Rest}`>]: string }
    : S extends `${infer Pre}/:${infer Param}`
    ? { [K in Param]: string }
    : {};

type R2 = Params<'/users/:userId/posts/:postId'>;
// { userId: string; postId: string }
```

This is how libraries like tRPC, Hono, and TanStack Router build typed routing without codegen.

---

## Utility Types Reference

Built into TypeScript; memorize the common ones.

| Utility | What it does |
|---|---|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties `readonly` |
| `Pick<T, K>` | Subset of properties (`K extends keyof T`) |
| `Omit<T, K>` | All properties except K |
| `Exclude<T, U>` | Members of T not assignable to U |
| `Extract<T, U>` | Members of T assignable to U |
| `NonNullable<T>` | Excludes `null` and `undefined` |
| `Record<K, V>` | Object with keys K and values V |
| `ReturnType<F>` | Return type of function F |
| `Parameters<F>` | Tuple of parameter types of F |
| `ConstructorParameters<C>` | Tuple of constructor params |
| `InstanceType<C>` | Instance type of class C |
| `Awaited<T>` | Unwraps Promise (recursively) |
| `ThisParameterType<F>` | Type of `this` parameter of F |
| `OmitThisParameter<F>` | F with `this` parameter removed |

### Examples
```ts
type User = { id: string; email: string; password: string };

type UserDTO    = Omit<User, 'password'>;
type UserUpdate = Partial<Omit<User, 'id'>>;
type UserMap    = Record<string, User>;

async function getUser() { return { id: '1', email: 'a@b.com' }; }
type Got = Awaited<ReturnType<typeof getUser>>;   // { id: string; email: string }
```

---

## Branded / Nominal Types

TypeScript is structurally typed. Two types with identical shape are interchangeable — even when they shouldn't be:

```ts
type UserId = string;
type OrderId = string;

function getUser(id: UserId): User { /* ... */ }

const orderId: OrderId = 'ord_123';
getUser(orderId);   // ❌ No error — both are just `string`
```

**Branding** simulates nominal typing by tagging a type with a phantom property:

```ts
type Brand<T, B> = T & { __brand: B };
type UserId  = Brand<string, 'UserId'>;
type OrderId = Brand<string, 'OrderId'>;

const asUserId  = (s: string): UserId  => s as UserId;
const asOrderId = (s: string): OrderId => s as OrderId;

function getUser(id: UserId) { /* ... */ }

getUser(asUserId('u_42'));    // ✅
getUser(asOrderId('o_42'));   // ❌ Argument of type OrderId not assignable to UserId
```

Use brands for:
- IDs that should not be mixed (`UserId` vs `OrderId` vs `ProductId`)
- Validated values (`Email`, `URL`, `PositiveNumber`) that have already passed runtime validation
- Currency amounts in different units (`USDCents`, `EURCents`)
- Trusted vs untrusted strings (`SafeHtml` vs `string`)

The runtime cost is zero. The compile-time safety is enormous.

---

## Variance & Function Types

### Covariance, contravariance, bivariance
- **Covariant** — subtype where supertype expected (return positions, readonly properties)
- **Contravariant** — supertype where subtype expected (function parameters)
- **Bivariant** — both directions allowed (TypeScript's default for *method* parameters)

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };

const dogs: Dog[] = [{ name: 'Rex', breed: 'Lab' }];
const animals: Animal[] = dogs;  // ✅ covariant — Dog[] assignable to Animal[]

const dogHandler: (d: Dog) => void = (d) => console.log(d.breed);
const animalHandler: (a: Animal) => void = (a) => console.log(a.name);
const slot1: (d: Dog) => void = animalHandler;  // ✅ contravariant — Animal handler accepts Dogs
```

### `strictFunctionTypes`
Enabled with `strict`. Enforces parameter contravariance for function types — but *not* for methods (that's a known historical inconsistency for ergonomic reasons).

### `strictNullChecks` is the variance flag that matters most
You won't hand-write variance often, but the compiler relies on it for every assignment. The flag that catches the most bugs is `strictNullChecks` (included in `strict`).

---

## Type-Only Imports & Module Resolution

### `import type` and `verbatimModuleSyntax`
```ts
import type { User } from './types';   // erased at compile time
import { fetchUser } from './api';     // runtime import
```

`verbatimModuleSyntax: true` *requires* `import type` for type-only imports. This eliminates ambiguity for bundlers and Node's ESM resolver, and prevents accidental runtime imports of types-only modules.

### `moduleResolution`
| Value | When |
|---|---|
| `Bundler` | Vite, Webpack, esbuild, Rollup — all modern bundlers. The right default in 2026. |
| `NodeNext` / `Node16` | Pure Node.js (ESM-aware) without a bundler |
| `Node` | Legacy CJS-only Node projects |
| `Classic` | Don't use |

`Bundler` is the modern default and matches what your build tool actually does.

### Package exports & conditional exports
Modern packages use `exports` in `package.json` to declare entry points per condition (`import` vs `require`, `node` vs `browser`, `types`, `default`). Mismatched configurations between your package and consumers' bundlers cause "the type is `any`" mysteries. Read [Andrew Branch's articles on the matter](https://github.com/microsoft/TypeScript/wiki/Performance) when this bites.

---

## Module Augmentation & Declaration Merging

### Declaration merging
Multiple `interface` declarations with the same name combine:
```ts
interface User { id: string }
interface User { email: string }
// User: { id: string; email: string }
```

Useful when extending types you don't own.

### Module augmentation
Add types to existing modules:
```ts
// Add `user` to Express's Request
declare module 'express-serve-static-core' {
  interface Request {
    user?: { id: string; role: string };
  }
}
```

### Global augmentation
```ts
declare global {
  interface Window {
    analytics: { track: (event: string) => void };
  }
}

window.analytics.track('page_view');   // ✅ typed
```

Use augmentation sparingly. Each `declare module` block is a contract you've made with a third-party library; if they change, you fix.

---

## React with TypeScript

### Component props
```tsx
type ButtonProps = {
  variant: 'primary' | 'secondary';
  onClick: () => void;
  children: React.ReactNode;
  disabled?: boolean;
};

export function Button({ variant, onClick, children, disabled }: ButtonProps) {
  return <button className={variant} onClick={onClick} disabled={disabled}>{children}</button>;
}
```

### Don't use `React.FC` (mostly)
The community has largely moved away from `React.FC` because of its handling of `children` and `defaultProps`. Use a plain function with typed props.

### `forwardRef`
```tsx
type InputProps = { label: string };

export const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ label }, ref) => (
    <label>{label}<input ref={ref} /></label>
  )
);
```

In React 19+, `ref` is just a prop — `forwardRef` is no longer required.

### Polymorphic components (`as` prop)
The "render as another tag/component" pattern. The "right" way is involved:

```tsx
type AsProp<C extends React.ElementType> = { as?: C };
type PolymorphicProps<C extends React.ElementType, Props = {}> =
  Props & AsProp<C> & Omit<React.ComponentPropsWithoutRef<C>, keyof Props | 'as'>;

function Box<C extends React.ElementType = 'div'>(
  { as, ...rest }: PolymorphicProps<C>
) {
  const Component = as || 'div';
  return <Component {...rest} />;
}

<Box as="a" href="/home">Home</Box>;   // typed correctly
```

Most product code can lean on libraries that have already solved this (Radix, React Aria, MUI). Roll your own only if you're building a design system.

### Generic components
```tsx
type SelectProps<T> = {
  options: T[];
  selected: T;
  onChange: (v: T) => void;
  getLabel: (v: T) => string;
};

function Select<T>({ options, selected, onChange, getLabel }: SelectProps<T>) {
  return (
    <ul>
      {options.map(o => (
        <li key={getLabel(o)} onClick={() => onChange(o)}>
          {getLabel(o)}
        </li>
      ))}
    </ul>
  );
}

<Select<User>
  options={users}
  selected={users[0]}
  onChange={setUser}
  getLabel={u => u.name}
/>
```

### Event handlers
```tsx
const onClick: React.MouseEventHandler<HTMLButtonElement> = (e) => { /* ... */ };
const onChange: React.ChangeEventHandler<HTMLInputElement> = (e) => { /* ... */ };
```

### Custom hooks
```tsx
function useToggle(initial = false): [boolean, () => void] {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []);
  return [value, toggle];
}
```

Return tuples for hooks the consumer destructures positionally; return objects when there are 3+ returns or when names matter more than position.

### Styling type integration
Tailwind + `class-variance-authority` (CVA) gives you typed component variants:
```ts
const button = cva('rounded-md', {
  variants: {
    intent: { primary: 'bg-blue-500', secondary: 'bg-gray-200' },
    size: { sm: 'text-sm', lg: 'text-lg' },
  },
});

type ButtonVariants = VariantProps<typeof button>;
```

---

## Runtime Validation & The Type/Value Boundary

Types disappear at compile time. Anything that crosses a runtime boundary — network responses, environment variables, query parameters, `localStorage`, file contents, user input — must be validated at runtime.

### The trio of choices

| Library | Strengths |
|---|---|
| **Zod** | Most popular; great DX; strong inference; large ecosystem |
| **Valibot** | Tree-shakeable; bundle-size-conscious; modular schema |
| **ArkType** | Closer to TS syntax; very expressive; performance focus |
| **io-ts** | FP-leaning, Effect-style codecs; smaller community |
| **Yup** | Older, form-validation-leaning; widely used |
| **`type-fest` + `valita`** | Smaller, surgical alternatives |

### Pattern: schema-first
```ts
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  age: z.number().int().nonnegative(),
});

type User = z.infer<typeof UserSchema>;   // derived type — single source of truth

export async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  return UserSchema.parse(await res.json());   // validates AND narrows
}
```

The schema is the source of truth; the type is derived. One change, both stay in sync.

### Where to validate
- **Network responses** — every `fetch().json()` should `.parse()`
- **Environment variables** — at app startup; fail fast with a clear error
- **Query/path/form parameters** — at the API handler entry
- **Untrusted serialized data** (`JSON.parse`, `localStorage`, postMessage)

Inside the trust boundary, types alone are enough.

---

## Type-Level Programming

The art of writing programs that the type checker evaluates. Useful for libraries; usually overkill for application code.

### Tuple manipulation
```ts
type Reverse<T extends any[]> = T extends [infer Head, ...infer Tail]
  ? [...Reverse<Tail>, Head]
  : [];
type R = Reverse<[1, 2, 3]>;   // [3, 2, 1]
```

### Recursion limits
TypeScript caps recursion depth (~1000 levels). Most type-level programs hit this for non-trivial inputs. **Lean toward simpler types unless complexity is the point.**

### When type-level programming earns its keep
- Library APIs (form validators, query builders, route definitions)
- DSLs that benefit from compile-time correctness
- Educational exercises (type-challenges, Type Hero)

### When it doesn't
Application code. If your app feature requires recursive conditional types, you're writing the wrong thing.

> **Worth knowing:** the TypeScript team's [Performance wiki page](https://github.com/microsoft/TypeScript/wiki/Performance) is the official guide to keeping types fast. Read it before going deep.

---

## TypeScript Performance

The compiler is fast, but unbounded type complexity is one of the few things that can slow it down dramatically.

### Symptoms of a slow project
- `tsc --noEmit` over 10 seconds for a small change
- VSCode's IntelliSense lag (red squigglies appear seconds after typing)
- "Type instantiation is excessively deep and possibly infinite" errors

### Common causes
- **Massive union types** (`type Foo = 'a' | 'b' | ... | 'zzz'` from a generated source)
- **Deep recursive types**
- **Conditional types over large unions** (distribution explodes)
- **Inferred types that walk too far** (`Awaited<Awaited<Awaited<...>>>`)
- **Dependency declaration files** (mitigate with `skipLibCheck`)

### Optimizations
1. **Add return-type annotations** on exported functions. Inference becomes a transitive cost; explicit annotation breaks the chain.
2. **Use `interface` over `type` for object shapes in hot paths** — slightly faster cache.
3. **`skipLibCheck: true`** if not already.
4. **Project references** — split a monorepo into independently-built TS projects. Build only what changed.
5. **Profile** with `tsc --extendedDiagnostics` and `tsc --generateTrace ./trace`. Open the trace in [`@typescript/analyze-trace`](https://github.com/microsoft/typescript-analyze-trace) or `chrome://tracing`.
6. **Type-coverage tools** — [`type-coverage`](https://github.com/plantain-00/type-coverage), `ts-prune`, `knip` — find dead exports and `any` leaks.

### tsserver vs. tsc
The IDE uses `tsserver`, which has different performance characteristics from `tsc`. If only the IDE is slow, restart the TS server (Cmd+Shift+P → "Restart TS Server") before assuming a code problem.

---

## Migration Strategies (JS → TS)

Migrating an existing JS codebase to TypeScript is a project, not a sprint.

### The phased approach
1. **Add `tsc` with `allowJs: true`, `checkJs: false`, `noEmit: true`** in CI — type-checks types but doesn't enforce on JS.
2. **Add `// @ts-check` comments** to high-value JS files. Get type-checking in JSDoc form for free.
3. **Rename `.js` → `.ts` file by file**, starting with leaf modules (no imports from JS consumers).
4. **Enable strict flags one at a time** — `noImplicitAny` first, then `strictNullChecks`, etc.
5. **Add CI gates** to prevent backsliding (`type-coverage` thresholds, no new `any`).

### Auto-conversion tools
- **`ts-migrate`** (Airbnb) — automated batch conversion with sensible defaults
- **`@typescript-tools/type-coverage`** — track adoption progress

### What to expect
- 1–6 months for a meaningful codebase
- Adoption isn't 100% on day one; aim for 90%+ within the first year
- The migration is sociotechnical — the hardest changes are the ones that surface old design mistakes the team has been avoiding

---

## Tooling Landscape

### TypeScript itself
- **`tsc`** — the compiler, type checker, and language server backend
- **TypeScript Project References** — multi-package monorepo support
- **`tsserver`** — backs IDE features

### Linting
- **typescript-eslint** — required for any TS codebase. Type-aware rules (rules requiring `parserOptions.project`) catch real bugs but are slower.
- **`@typescript-eslint/eslint-plugin`** key rules:
  - `no-floating-promises` — catches forgotten `await`
  - `no-misused-promises` — passing async functions where sync expected
  - `no-unnecessary-type-assertion`
  - `prefer-readonly-parameter-types`
  - `consistent-type-imports`
  - `strict-boolean-expressions`
- **Biome** — Rust-based, fast, growing TS support; emerging alternative to ESLint+Prettier

### Bundlers / runtimes (TS-aware)
- **Vite**, **esbuild**, **swc** — fast, do not type-check (run `tsc --noEmit` separately)
- **tsx**, **ts-node** — Node TS execution
- **Bun** — runs TS natively
- **Deno** — runs TS natively, with built-in type checking

### Code-quality
- **`type-coverage`** — % of your codebase that's typed (not `any`)
- **`ts-prune` / `knip`** — find unused exports
- **`expect-type`**, **`tsd`** — test type-level behavior
- **`madge`**, **`dependency-cruiser`** — visualize import graphs, enforce architecture

### Runtime validation
- **Zod**, **Valibot**, **ArkType**, **io-ts**, **Yup** (covered above)
- **TypeBox** — generates JSON Schema and TS types from one source

### IDE
- VSCode + TypeScript extension is the de facto standard
- **TypeScript Hero** for import management
- **Pretty TypeScript Errors** — humanizes long error messages
- **TypeScript Vista** / **Code Navigation** — for type hierarchies in large codebases

### AI-assisted typing
- **GitHub Copilot**, **Cursor**, **Codeium**, **Claude / ChatGPT** — surprisingly good at writing precise TS types when you describe the constraint clearly. Use them for utility-type wrangling.

---

## Common Anti-Patterns

### 1. Reaching for `any`
Every `any` is a hole. If you must, use `unknown` and narrow.

### 2. Type assertions over type guards
```ts
const user = data as User;   // Trust me bro
```
Asserts truth without verification. Replace with a type guard or a Zod parse.

### 3. Disabling strict mode
"It's annoying" is not a reason. Disabling strict turns TypeScript into typed-flavored JavaScript.

### 4. Sprinkling `@ts-ignore` and `@ts-expect-error`
- `@ts-ignore` permanently silences the next line — use *only* with a comment explaining why.
- `@ts-expect-error` is better — it requires a real error on the line; turns into an error if the issue is fixed. Always prefer.
- Track them in CI; cap the count.

### 5. Type guards that lie
```ts
function isUser(x: unknown): x is User {
  return typeof x === 'object';   // ❌ Says nothing about User-ness
}
```
The compiler trusts you. If your guard is wrong, you'll have hard-to-debug runtime errors.

### 6. Overusing `enum`
TypeScript `enum`s have weird runtime semantics (numeric enums are bidirectional maps), generate emit code, and are inconsistent with `const enum` (which has its own footguns). **Prefer string literal unions + `as const` objects:**
```ts
// ❌
enum Status { Active, Inactive }

// ✅
const Status = { Active: 'active', Inactive: 'inactive' } as const;
type Status = typeof Status[keyof typeof Status];
```

### 7. Generics with no constraint
`<T>` with no constraint is rarely what you want. Most generics need at least `T extends ...` to be useful.

### 8. Type gymnastics for what should be a runtime check
If your type-level program needs 80 lines to express, the runtime equivalent is 5 lines and a Zod schema. Pick the right level.

### 9. Inferring everything everywhere
Inference is great for locals; explicit return types are better at API boundaries. They speed up the compiler and document the contract.

### 10. Hand-writing types from APIs
Use code-gen (OpenAPI Generator, GraphQL Codegen, Prisma) when there's a schema. Hand-writing drifts from reality the day after you commit.

### 11. Ignoring `unknown` in catch
```ts
try { ... } catch (e) {
  console.log(e.message);   // ❌ e is unknown, .message could fail
}
```
With `useUnknownInCatchVariables` (in `strict`), `catch (e)` is `unknown`. Narrow first.

### 12. Treating types as documentation rather than constraints
A type that allows everything ({} or `Record<string, any>`) documents a shape but doesn't constrain anything. Aim for types that *would catch* bad inputs.

### 13. `Function` and `Object` types
`Function` is `any`-flavored and unsafe. Use `(...args: any[]) => any` (intentional) or, better, a precise signature. `Object` is similarly weak; use `object` (lowercase) or `Record<string, unknown>`.

### 14. Mixing `interface` and `type` randomly
Pick a project convention and lint for it.

### 15. Deeply nested optional chains as a substitute for narrowing
```ts
user?.profile?.settings?.theme   // sometimes the right answer
```
But if you're doing this everywhere, your *types* are wrong — model the union explicitly.

---

## Quick Reference Checklist

### Configuration
- [ ] `strict: true` and all flags within it
- [ ] `noUncheckedIndexedAccess: true`
- [ ] `exactOptionalPropertyTypes: true`
- [ ] `noImplicitOverride: true`, `noImplicitReturns: true`
- [ ] `verbatimModuleSyntax: true`, `isolatedModules: true`
- [ ] `skipLibCheck: true`
- [ ] `moduleResolution: "Bundler"` (or `NodeNext` for non-bundled Node)
- [ ] CI runs `tsc --noEmit` on every PR

### Code patterns
- [ ] No bare `any` in source code
- [ ] `unknown` at trust boundaries; narrow with type guards or runtime schemas
- [ ] Discriminated unions for state with multiple possible shapes
- [ ] Exhaustive-`never` on all switch statements over unions
- [ ] Branded types for IDs and validated values
- [ ] `as const` + indexed-access for deriving unions from objects
- [ ] `satisfies` for config objects
- [ ] Return-type annotations on exported functions

### Runtime safety
- [ ] Zod (or equivalent) validates every untrusted input
- [ ] One source of truth for shape: schema → derived type
- [ ] Environment variables validated at startup
- [ ] API responses parsed before use

### Tooling
- [ ] typescript-eslint with type-aware rules
- [ ] `@ts-expect-error` over `@ts-ignore`; counted in CI
- [ ] `type-coverage` threshold gating
- [ ] `knip` / `ts-prune` for dead exports
- [ ] Source-of-truth code-gen for API boundaries (OpenAPI / GraphQL / Prisma / tRPC)

### React-specific
- [ ] Function components with typed props (no `React.FC`)
- [ ] Generic components where the API benefits
- [ ] Polymorphic components from a vetted library
- [ ] Custom hooks return tuples for ≤2 values, objects otherwise
- [ ] Type-safe styling (CVA, Tailwind, vanilla-extract)

---

## Further Reading

### Books
- **Dan Vanderkam — *Effective TypeScript* (2nd ed., 2024)** — 83 specific items; the most pragmatic book on TS. Required reading.
- **Boris Cherny — *Programming TypeScript*** (O'Reilly) — comprehensive reference
- **Marius Schulz — Email newsletters and articles** — deep dives on individual features

### Free online
- **[The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)** — official, exceptionally well-written
- **[Total TypeScript by Matt Pocock](https://www.totaltypescript.com/)** — paid courses + a wealth of free content; the most influential modern TS educator
- **[Type-Challenges](https://github.com/type-challenges/type-challenges)** — type-level puzzles, easy → extreme
- **[TypeHero](https://typehero.dev/)** — gamified type-level puzzles
- **[TypeScript Performance wiki page](https://github.com/microsoft/TypeScript/wiki/Performance)** — the official tuning guide
- **[Stefan Baumgartner — *TypeScript in 50 Lessons*](https://typescript-book.fettblog.eu/)** — short, free
- **[Anders Hejlsberg's talks](https://www.youtube.com/results?search_query=anders+hejlsberg+typescript)** — the language designer; deep insights into trade-offs

### Newsletters
- **TypeScript Weekly** — community-curated
- **Bytes by ui.dev** — broader, but TS-heavy

### Specific topics
- **Module resolution & ESM**: TypeScript team's official write-ups; Andrew Branch's articles
- **Performance**: TypeScript Performance wiki, `analyze-trace` README
- **Type-level programming**: Type-Challenges + Mateusz Burzyński's `ts-toolbelt` source code
- **React + TS**: [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

### Cross-references in this repo
- See [Web Testing Strategy](web-testing-strategy.md) for type-level test tooling (`expect-type`, `tsd`).
- See [API Design](api-design.md) for end-to-end-typed RPC patterns (tRPC, OpenAPI codegen).
- See [Web Application Security](web-application-security.md) for runtime validation as a security boundary.
- See [Glossary](glossary.md) for term definitions.

---

## Key terms

This guide is the canonical home for these glossary entries: **`any`**, **`unknown`**, **`never`**, **`as const`**, **`satisfies`**, **Branded Type**, **Conditional Type**, **Discriminated Union**, **Generic**, **`infer`**, **Mapped Type**, **Template Literal Type**, **Strict Mode**, **Structural Typing**, **Type Guard / Type Predicate**, **Type Narrowing**, **Utility Types**, **`noUncheckedIndexedAccess`**, **`verbatimModuleSyntax`**, **Zod / Valibot / ArkType**, **Module Augmentation**, **Declaration Merging**, **Distributive Conditional Type**, **Exhaustive Check**. See the [glossary](glossary.md) for definitions and the rest of the index.

---

> **Closing thought:** TypeScript at its best disappears. The types do their work — autocomplete fires, refactors propagate, bad code refuses to compile — and the language gets out of the way of the actual program you're building. The teams I've watched succeed with TS aren't the ones who wrote the most clever generic types; they're the ones who set strict mode on day one, validated at every boundary, and treated `any` as a leak to fix rather than a tool to reach for. The reward is a codebase that resists rot for years longer than its untyped equivalent — and an engineer experience that makes adding a new feature feel like editing a contract, not like throwing code over a wall and hoping. Get the foundations right, lean on inference, and let the compiler be the colleague that never gets tired of catching the same bug.
