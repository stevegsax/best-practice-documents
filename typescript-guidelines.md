# TypeScript guidelines

## General Guidelines

- Use `pnpm` to manage dependencies and workspaces **on Node**. It is strict by default — no phantom dependencies, so a package you didn't declare can't accidentally resolve — and its content-addressed store keeps installs fast and reproducible against a committed `pnpm-lock.yaml`. (Deno and Bun manage dependencies through their own tooling, covered under *Scripts and helpers*.)
- **Write runtime-agnostic code by default.** Target Web-standard APIs (`fetch`, `Request`/`Response`, `ReadableStream`, `URL`, `URLSearchParams`, `TextEncoder`/`TextDecoder`, `structuredClone`, `AbortController`, `crypto.randomUUID()`, `crypto.subtle`, `performance.now()`) so the same code runs on Node, Deno, Bun, and the browser. Reach for a runtime-specific built-in (`node:fs`, `Deno.*`, `Bun.*`, `process.env`, `__dirname`) only in the shell, behind an interface — never in the core. "Works on my runtime" is a portability bug, not a fact.
- **ESM only.** `"type": "module"`, a modern `module`/`moduleResolution` (`nodenext` or `bundler`), and explicit import extensions where the resolver requires them (`./trades.js` from a `.ts` source under `nodenext`). No CommonJS, no `require`, no `namespace` in new code; use `import.meta.url`, not `__dirname`.
- Correctness and testability are more important than performance.
- **Pin the toolchain.** Record the versions this code is built and validated against — TypeScript, the runtime (Node/Deno/Bun), and the test runner — so a build is reproducible rather than dependent on whatever floats in. These guidelines were written against TypeScript 6, Node 24–25, and Vitest 4.
- Where possible, adopt a `functional programming` design philosophy where functions are **pure** — deterministic and free of side effects — and side effects are handled separately.
- Apply the "Functional Core / Imperative Shell" pattern.
- Prefer `map`, `filter`, `reduce`, and small composed functions over imperative loops with mutation. Model value types as plain `readonly` data plus functions, not stateful classes.

## Scripts and helpers

- **A helper should run with no build step.** On modern Node, run a standalone `.ts` file directly with `node script.ts` — type stripping is on by default (stable since Node 24.12 / 25.2), so there is no dependency at all; `tsx script.ts` (or `node --import tsx script.ts`) remains useful on older Node or when you need more than plain type erasure. Deno and Bun execute TypeScript directly (`deno run script.ts`, `bun script.ts`). There is no `tsc` artifact to forget about and no stale `dist/` to debug against. Native stripping accepts only erasable syntax — no `enum`, no runtime `namespace` — which is exactly the subset `erasableSyntaxOnly` enforces (see *Type system discipline*).
- **Declare dependencies explicitly and pin them.** The PEP 723 analog — a script self-contained and reproducible wherever it lands — is closest in Deno, where a script pins its imports to versioned specifiers (`jsr:@std/...@1.2.3`, `npm:zod@3.23.8`) recorded in a committed `deno.lock`. On Node, keep a local `package.json` + `pnpm-lock.yaml` next to the script rather than relying on globally installed or ambient packages. Either way, the dependency set lives with the script, not in an assumed environment.

## Functional Core / Imperative Shell — practice rules

The pattern survives contact with everyday development only when these rules are followed in the code itself. Several of them are contrary to mainstream TypeScript defaults; treat them as deliberate departures.

### Module structure

- **Named exports, not `default`.** Default exports rename inconsistently across import sites, hurt automated refactoring and "find references," and obscure tree-shaking. One conceptual module per file; export the public surface by name.
- **Imports go inward, never reverse.** Pure modules MUST NOT import from modules that do I/O, hold state, or call runtime/framework APIs. If you need to, the boundary is in the wrong place — push the I/O out, not the import in. Enforce this mechanically — it is the one architectural invariant otherwise left to discipline: `dependency-cruiser`, `eslint-plugin-import`'s `no-restricted-paths`, or `eslint-plugin-boundaries` can fail the build when a core module imports from the shell.
- **No import-for-side-effect.** `import "./register-routes.js"` and barrel files (`index.ts`) that exist only to run registration code are an anti-pattern, even when intentional. Import-time side effects depend on the importer running at the right moment and on module evaluation order, and they break silently. Use the framework's own discovery mechanism (a passed registry, explicit `register(app)` calls in the shell, plugin specs) instead. Barrels that merely re-export are tolerable but invite circular imports — keep them shallow.
- **`import type` for type-only imports**, and enable `verbatimModuleSyntax`. A type import must never pull a module in at runtime or trigger its side effects; making the distinction explicit guarantees it.

### Immutability by default

- **`readonly` everywhere.** `readonly` fields on object types, `readonly T[]` (or `ReadonlyArray<T>`), `ReadonlyMap`, `ReadonlySet` in signatures. `as const` for literal data. Produce new values with spread (`{ ...session, expires }`) rather than mutating in place.
- **`readonly` is compile-time and shallow.** It erases at runtime (no `Object.freeze`) and does not protect nested objects — `readonly outer: { inner: number }` still permits `x.outer.inner = …`. Reach for `Object.freeze`/`as const` or a deep-readonly type where a *runtime* guarantee matters. And note that `Date` is itself mutable (`d.setHours(…)`), so a `readonly expires: Date` is not truly immutable; prefer `Temporal` (shipping in engines) or treat `Date` as a known wart and never mutate it in place.
- **`ReadonlyArray<T>` / `Iterable<T>` over `T[]` in parameters.** A `readonly` parameter signals a read-only contract and lets callers pass any compatible value; returning `readonly T[]` tells callers they must not mutate the result. Mutable `T[]` implies "callers may mutate me."
- **Prefer immutable value types over classes.** Model data as a `readonly` object type and transform it functionally. Don't reach for Immer or a persistent-data-structure library until shallow spread genuinely stops scaling; for nested updates that get unwieldy, Immer's `produce` is the justified exception, not the default.
- **`const` by default; `let` is a smell to justify; never `var`.** Reach for a `map`/`reduce`/`filter` pipeline before a `let` accumulator. A `let` is acceptable only when the imperative form is genuinely clearer, and it never escapes the function it lives in.

### Dependency injection — by parameter, not by container

- **Pass dependencies as parameters; do NOT use a DI container.** No InversifyJS, no tsyringe, no framework auto-wiring for the core. (This is also why this guidance does not adopt Effect: its `Context`/`Layer` system is a DI container, which cuts against this rule.) The shell wires concrete dependencies at the outermost boundary and passes them down through typed function signatures.
- **No module-level singletons for I/O clients.** Database pools, `fetch` wrappers, object-storage and queue clients are created in the shell at startup and passed explicitly. Tests substitute a test double the same way production wires the real client.
- **Pure functions take only the values they need, not a whole `Config`.** A function that takes the entire `Config` object is hiding its real dependencies — pass `config.sessionLifetimeSeconds`, not `config`.
- **Depend on a structural `interface`, not a concrete class.** TypeScript's structural typing is the `Protocol` equivalent: declare the narrow shape the core needs (`interface Clock { now(): Date }`) and let any object satisfy it. Tests pass a literal; production passes the real adapter.

### Hidden state — none

- **No module-level mutable state.** A `let cache = …` or `const items: T[] = []` at module scope is a global; it is not load-bearing for any code path you care about. If something seems to need it, you've conflated a value (config) with a service (something stateful).
- **No memoization of impure functions.** Memoizing an I/O call assumes referential transparency it doesn't have. Caching results of effectful work is a separate concern, handled deliberately in the shell.
- **Top-level code must not perform I/O.** `const client = new Client()` or a `fetch` at module scope runs on import and couples loading to side effects. Stateful resources (pools, caches, sessions) are created once in the shell and passed inward; the core sees only the values it needs from one.

### Type system discipline

- **`strict: true`, and then more.** Also enable `noUncheckedIndexedAccess`, `noPropertyAccessFromIndexSignature`, `exactOptionalPropertyTypes`, `noImplicitOverride`, `noFallthroughCasesInSwitch`, `noImplicitReturns`, `isolatedModules`, `verbatimModuleSyntax`, and `erasableSyntaxOnly` — the last rejecting exactly the non-erasable constructs (`enum`, runtime `namespace`, parameter properties) and so mechanically enforcing the no-`enum`/no-`namespace` rules this document otherwise states only in prose. Treat new type errors as build breaks, not warnings.
- **Pin `target`, `lib`, and `module`.** TypeScript 6 floats these defaults — `target` tracks the current year (e.g. `es2025`), with `module: esnext` and `types: []` — so an unpinned project emits silently different JavaScript across compiler upgrades, unacceptable where builds must be reproducible. Pin them explicitly; a pinned modern `target` (≥ `es2022`) also keeps `instanceof` reliable for `Error` subclasses, which the catch-narrowing rule below depends on.
- **Lint enforces the architecture.** Use `typescript-eslint` with a type-checked config (or Biome where its rule set suffices) and turn on the rules that encode this document: `no-floating-promises` and its partner `no-misused-promises`, `await-thenable`, `require-await`, `no-explicit-any`, `no-non-null-assertion`, `switch-exhaustiveness-check`, `consistent-type-imports`, `no-unnecessary-condition`, `no-confusing-void-expression`, `prefer-readonly`, `only-throw-error` (the enforcer for "never throw a string", below), and `no-unsafe-*`. A rule that fires is the guideline talking.
- **`unknown` over `any`.** `any` switches the checker off and is contagious. If you must use it, leave a comment explaining why; the lint rule should otherwise forbid it.
- **Avoid type assertions.** `as` and the non-null `!` defeat the checker and are how "impossible" runtime errors happen. Narrow with type guards or parse with a schema instead; reserve `as` for the validated boundary where you've just proven the shape. Use `satisfies` to check a value against a type without widening it.
- **Branded (nominal) types for IDs.** TypeScript is structural, so `UserId` and `DocumentId` are interchangeable strings unless you brand them. Make them distinct so a typo that "happens to compile" can't:

  ```ts
  declare const brand: unique symbol;
  type Brand<T, B extends string> = T & { readonly [brand]: B };
  type UserId = Brand<string, "UserId">;
  type DocumentId = Brand<string, "DocumentId">;
  ```

  Construct them only through a smart constructor that validates, so the brand means "checked," not just "asserted."
- **Brand units and validated values, not just IDs.** The same technique stops you adding basis points to a price or passing a notional where a quantity was expected — the "typo that happens to compile" applied to the firm's core domain. Construct through a validating smart constructor that returns a `Result`, so the one `as` lives at that boundary and the brand means "checked":

  ```ts
  type Price = Brand<number, "Price">;
  type Bps = Brand<number, "Bps">;
  type Qty = Brand<number, "Qty">;

  // The brand means "checked", not "asserted": construct only here.
  function toPrice(n: number): Result<Price, "not-a-price"> {
    return Number.isFinite(n) && n >= 0
      ? { ok: true, value: n as Price } // sanctioned `as`: shape just proven
      : { ok: false, error: "not-a-price" };
  }
  ```

- **Closed sets are unions of string literals, not `enum`.** `type Status = "open" | "closed" | "settled"` (optionally backed by an `as const` object) avoids `enum`'s runtime emit and nominal quirks, and `const enum` is incompatible with `isolatedModules`. Use `Literal`-style unions and `as const` lookup tables instead.

### I/O, runtime APIs, and async at the boundary

- **`async` / `Promise` implies I/O — it lives in the shell.** Pure-core functions are sync. This is contrary to codebases that make everything async; do not follow that lead, and don't propagate `async` farther than needed — once a function is async, every caller has to be too.
- **Runtime APIs are shell-only.** Filesystem, network, `process`/`Deno`/`Bun` globals, environment, clock, and console belong at the boundary. A pure command takes structured data and returns structured data with none of these in its body. When the core genuinely needs a capability, inject it as a narrow interface (`interface FileStore { read(key: string): Promise<Uint8Array> }`) and supply the runtime-specific adapter in the shell.
- **Parse at the edge, compute in the core.** Validate external data — from `fetch`, a file, `JSON.parse` — into a known typed shape once at the boundary with a schema (zod or valibot), then let the core trust the type. "Parse, don't validate." Where the boundary only needs to *run* a validator, depend on the **Standard Schema** interface (`~standard`, common to Zod, Valibot, and ArkType) rather than a specific library — the "depend on a structural interface" rule applied to validation, so one implementation can be swapped for another without touching call sites.
- **No floating promises.** Every promise is `await`ed or its rejection explicitly handled; `no-floating-promises` (and its partner `no-misused-promises`) enforces it. Thread an injected `AbortSignal` for cancellation rather than firing and forgetting. Concurrency that does I/O (`Promise.all` over `fetch` calls) is a shell concern — quarantine it there; pure transformations stay sync and trivially testable. Remember `Promise.all` rejects on the *first* failure and abandons the rest: where partial success is meaningful — fanning out to several venues or brokers — use `Promise.allSettled` and fold the outcomes, or `Promise.any` (which throws an `AggregateError` only when every promise rejects).

### Side effects: logging, time, randomness

- **Logging is a side effect.** It belongs in the shell. Pure functions return values; the shell (with a structured logger such as pino) decides what to log about them. No `console.log` in business logic — ever.
- **Time, randomness, UUIDs, hostname, and environment are pseudo-I/O.** `new Date()`, `Date.now()`, `Math.random()`, `crypto.randomUUID()`, and `process.env` produce different values each run. Accept them as parameters in the core (`now: Date`, or an injected `Clock`/`() => string`) and produce them at the shell. Tests then pass deterministic values with no fake timers and no monkeypatching.

### Configuration

- **One `Config` type, parsed once at startup.** Read the environment at the boundary through a small runtime adapter (`process.env`, `Deno.env.toObject()`, or `import.meta.env` differ per runtime), validate it with a schema, and freeze it into a `readonly Config`. Pass concrete values inward.
- **Read environment variables only in the config loader.** Don't scatter `process.env.X` through the code; every value flows from the single validated `Config`.
- **Fail fast on missing required config.** A schema `.parse` that throws at startup is correct here — a missing `DATABASE_URL` should crash boot, not surface as a 500 ten minutes later. Use `safeParse` only when you deliberately want to handle the failure as a value.

### Error handling

- **The failure channel is untyped — that is the core problem.** A `Promise<T>` says nothing about what it rejects with, and a `throw` is invisible to the type system; that is exactly why a caught value is `unknown`. When the *set* of failures is part of a function's contract, name it — a `Result<T, E>` that names `E` (below) turns "what can go wrong here?" from tribal knowledge into a type the caller must handle.
- **Domain-specific `Error` subclasses, not strings or bare `Error`.** Set a stable `name`, attach a discriminant, and pass the originating error via the `cause` option (`new ParseError(msg, { cause })`). The type IS the error category; the message carries the human detail. Never `throw` a string or a plain object.
- **Errors don't `JSON.stringify`.** `message`, `stack`, and `cause` are non-enumerable, so `JSON.stringify(err)` yields `{}`. Log through pino's `err` serializer (or serialize `{ name, message, stack, cause }` explicitly), and never interpolate a bare error into a string — that drops its `cause` chain.
- **Pure-core functions rarely throw — encode failures in return types.** A discriminated-union `Result` makes failure a value the caller must handle:

  ```ts
  type Result<T, E> =
    | { readonly ok: true; readonly value: T }
    | { readonly ok: false; readonly error: E };
  ```

- **`catch` is `unknown` — narrow before you handle.** With `useUnknownInCatchVariables` (on under `strict`), a caught value is `unknown`. Test it (`instanceof DomainError`) before using it, and re-throw what you don't recognize. An empty `catch {}` or one that swallows everything is "I don't know what fails here" — a design problem, not a coding one. Wrap only the failing call in `try`, not a whole pipeline.
- **`Result` types are a sometimes-tool.** Reach for the union above — or `neverthrow`'s `ok`/`err` with `.map`/`.andThen` when error composition matters (a pipeline of fallible steps, each failing differently). Use exceptions for one-off, genuinely exceptional conditions. Don't mix the two styles within a layer; pick one per layer. Branch on a `Result` or any tagged union with `ts-pattern`'s `match(...).exhaustive()`.
- **Schema results are `Result`s — adapt them at the edge.** A schema's `safeParse` returns a tagged success/failure; fold it onto the core's `Result` and map the library's error type to a domain one, so the core never sees a `ZodError`. (`.parse`, which throws, is for the fail-fast config path in *Configuration* — don't mix the throwing and value styles within one layer.)

  ```ts
  import { z } from "zod";

  const FillSchema = z.object({ sym: z.string().min(1), qty: z.number().int(), px: z.number() });
  type Fill = z.infer<typeof FillSchema>;

  // safeParse returns a tagged result; fold it onto the core's Result and map
  // the library error to a domain one, so the core never sees a ZodError.
  function parseFillAtEdge(raw: unknown): Result<Fill, { readonly kind: "invalid-fill"; readonly issues: readonly string[] }> {
    const r = FillSchema.safeParse(raw);
    return r.success
      ? { ok: true, value: r.data }
      : { ok: false, error: { kind: "invalid-fill", issues: r.error.issues.map((i) => i.message) } };
  }
  ```

### Make illegal states unrepresentable

- **Encode invariants in types.** If two fields can never be set together, model the union of valid states, not two `optional` fields that permit the illegal fourth combination.
- **Replace boolean/optional soup with a state union.** The canonical case: don't model an order as independent `filled`/`rejected` booleans plus optional `fillPx`/`reason` — that makes the impossible combinations (`filled && rejected`; a `fillPx` with no fill) just as constructible as the valid ones. Model the legal states directly, so the compiler rejects the rest:

  ```ts
  // Illegal: every combination is constructible, including the impossible ones.
  interface BadOrder {
    readonly filled: boolean;
    readonly rejected: boolean;
    readonly fillPx?: number;
    readonly rejectReason?: string;
  }

  // Legal states only: a fill price cannot exist without a fill, and an order
  // cannot be both filled and rejected.
  type Order =
    | { readonly status: "pending" }
    | { readonly status: "filled"; readonly fillPx: Price; readonly filledAt: Date }
    | { readonly status: "rejected"; readonly reason: string };
  ```

- **Discriminated unions + exhaustive `match`.** Tag each variant with a literal `kind`/`type` field and branch with a `switch` whose `default` calls `assertNever(x: never)`, or with `ts-pattern`'s `.exhaustive()`. Adding a variant then becomes a compile-time exhaustiveness error at every site that handled the old set:

  ```ts
  function assertNever(x: never): never {
    throw new Error(`unhandled variant: ${JSON.stringify(x)}`);
  }
  ```

- **Exhaustive lookup tables with `satisfies Record<Union, …>`.** When a branch is really a total mapping, a table checked against the union needs no runtime `default` at all — a missing or misspelled key is a compile error:

  ```ts
  type Side = "buy" | "sell";
  const sign = { buy: 1, sell: -1 } as const satisfies Record<Side, 1 | -1>;
  ```

- **Encode cardinality.** When "at least one" is part of the contract, say so in the type instead of checking `length` at runtime:

  ```ts
  type NonEmptyArray<T> = readonly [T, ...T[]];
  // The best bid of an empty ladder is meaningless — make it unrepresentable:
  declare function bestBid(ladder: NonEmptyArray<Price>): Price;
  ```

- **Branded types for IDs everywhere**, and `readonly` to forbid the mutation-driven illegal transition. Validate loose external input into the branded, precise shape once at the boundary (with a schema); the core is written against that shape and does not re-check it everywhere.

## Test strategy

The Functional Core / Imperative Shell commitment shapes every test. The pyramid has three layers — its mass is in pure unit tests because the architecture pushes most logic into the core, where testing is a literal-in / literal-out comparison.

- **Pure tests** — functional-core code. Conventionally under `test/unit/` (or co-located `*.test.ts`). No I/O, no mocks; the only "fixtures" are `test.each` case tables and `fast-check` property generators. They run in microseconds, so exhaustive parametrization and property-based testing are encouraged precisely because they're cheap here.
- **Integration tests** — imperative-shell code. Grouped separately and run against real dependencies: an ephemeral Postgres via `testcontainers`, real cloud buckets/queues isolated by a per-run prefix, a sandbox SMTP. NOT mocks of your own code.
- **End-to-end / BDD tests** — full-stack behavioral coverage. Test bodies drive the HTTP/CLI boundary, not service-layer internals.

Vitest is the runner. Rules that apply across layers:

- **Do not mock your own collaborators.** `vi.mock("./my-service.js")` in a route test almost always means either (a) it should be a real integration test, or (b) the logic worth testing is pure and belongs in a unit test of that pure function.
- **Mock only what's genuinely external.** Third-party services you don't control: `msw` for outbound HTTP, an in-memory test double behind your own injected interface for everything else. Prefer a real sandboxed resource over a mock.
- **Time, randomness, UUIDs, environment are pseudo-I/O.** Pass deterministic values into the core; supply the real source at the shell. Tests should not need `vi.useFakeTimers()` for core logic — if they do, the clock isn't injected.
- **Assert on observable behavior, not implementation.** Row written, file present, HTTP status and response body, the returned value. NOT `toHaveBeenCalledWith` on an internal collaborator, snapshot of a `toString()`, or which private method ran.

Anti-patterns to flag in review:

- `vi.mock` / `vi.spyOn` of internal modules and collaborators.
- `expect(fn).rejects.toThrow()` or `.toThrow()` with no argument — pin the specific error type and a message substring.
- Hardcoded production paths (use a temp directory).
- A pure-function unit test that needs more than a `test.each`-worth of setup — the function isn't pure; fix the function, not the test.
- Import-for-side-effect to register a route/plugin/step — use the framework's own discovery mechanism.
- A test that's "almost the same as that other test" — promote shared cases to a `test.each` table.
- A refactor that requires editing tests — either observable behavior changed (then it's not a refactor) or the test was coupled to implementation (fix the coupling, don't preserve it).

## Test safety

Tests MUST NOT write to production databases, config paths, or cache directories.

- Confine all filesystem writes to a temp directory (`fs.mkdtemp(os.tmpdir() + sep)`, or Vitest's per-test tmp helpers); never write outside it.
- Never hardcode XDG or home-relative production paths — `~/.local/share/`, `~/.config/`, `~/.cache/`. Let setup override these via env vars pointing into the temp dir.
- Use the project's test fixtures and injected interfaces rather than constructing real clients or database connections directly in a test.
- A global setup file (Vitest `setupFiles`) should refuse to run a destructive integration suite unless the active `DATABASE_URL` carries a test marker (`test`, `_dev`, `localhost`, `127.0.0.1`). The same principle applies to any other backing service with a production endpoint reachable from a dev machine.
- Type-annotate test helpers; they are code too.

### Worked example

A pure core module and its Vitest test. The test demonstrates: a deterministic pseudo-I/O parameter (`now`) instead of `new Date()`; a `test.each` case table; assertions on the returned structured value rather than its rendering; and an error path expressed as a `Result` value that the test pins exactly, rather than asserting only that something threw.

`src/trades.ts` — functional core, no I/O:

```ts
// Branded ID — structural typing won't let a bare string slip in.
declare const brand: unique symbol;
type Brand<T, B extends string> = T & { readonly [brand]: B };
export type SessionId = Brand<string, "SessionId">;

export interface Session {
  readonly id: SessionId;
  readonly expires: Date;
}

export interface Classified {
  readonly live: readonly Session[];
  readonly expired: readonly Session[];
}

// `now` is passed in (pseudo-I/O) so the core stays pure and tests are
// deterministic — no `new Date()` reaching into the clock.
export function classifySessions(sessions: readonly Session[], now: Date): Classified {
  return {
    live: sessions.filter((s) => s.expires > now),
    expired: sessions.filter((s) => s.expires <= now),
  };
}

export type Result<T, E> =
  | { readonly ok: true; readonly value: T }
  | { readonly ok: false; readonly error: E };

// Smart constructor for the SessionId brand above: validate, don't assert, so
// the only `as` is here at the boundary and the brand means "checked".
export function toSessionId(raw: string): Result<SessionId, "empty-session-id"> {
  return raw.length > 0 ? { ok: true, value: raw as SessionId } : { ok: false, error: "empty-session-id" };
}

export interface Fill {
  readonly sym: string;
  readonly qty: number;
  readonly px: number;
}

export type ParseError =
  | { readonly kind: "missing-field"; readonly field: "sym" | "qty" | "px" }
  | { readonly kind: "bad-number"; readonly field: "qty" | "px" };

// Empty/blank strings coerce to 0 via Number(), so guard them explicitly.
const num = (v: unknown): number | undefined =>
  typeof v === "number" ? v
  : typeof v === "string" && v.trim() !== "" ? Number(v)
  : undefined;

// Failures are values, not exceptions, in the core. (A real boundary would build
// `raw` through a schema — see "Parse at the edge" and "Error handling"; this
// hand-rolled parser exists to show a Result with a domain-specific error union.)
export function parseFill(raw: Record<string, unknown>): Result<Fill, ParseError> {
  const sym = raw["sym"];
  if (typeof sym !== "string" || sym.trim() === "")
    return { ok: false, error: { kind: "missing-field", field: "sym" } };
  if (raw["qty"] == null) return { ok: false, error: { kind: "missing-field", field: "qty" } };
  const qty = num(raw["qty"]);
  if (qty === undefined || !Number.isInteger(qty))
    return { ok: false, error: { kind: "bad-number", field: "qty" } };
  if (raw["px"] == null) return { ok: false, error: { kind: "missing-field", field: "px" } };
  const px = num(raw["px"]);
  if (px === undefined || !Number.isFinite(px))
    return { ok: false, error: { kind: "bad-number", field: "px" } };
  return { ok: true, value: { sym, qty, px } };
}
```

`test/trades.test.ts`:

```ts
import { describe, expect, test } from "vitest";
import { classifySessions, parseFill, toSessionId, type Session, type SessionId } from "../src/trades.js";

// Build ids through the real smart constructor, not a bare `as` cast.
const sid = (s: string): SessionId => {
  const r = toSessionId(s);
  if (!r.ok) throw new Error(r.error);
  return r.value;
};
const now = new Date("2026-05-16T12:00:00Z");

describe("classifySessions", () => {
  test.each([
    { name: "all live", sessions: [{ id: sid("a"), expires: new Date("2026-05-17T00:00:00Z") }], live: 1, expired: 0 },
    { name: "all expired", sessions: [{ id: sid("b"), expires: new Date("2026-05-15T00:00:00Z") }], live: 0, expired: 1 },
    {
      name: "mixed",
      sessions: [
        { id: sid("c"), expires: new Date("2026-05-17T00:00:00Z") },
        { id: sid("d"), expires: new Date("2026-05-15T00:00:00Z") },
      ],
      live: 1,
      expired: 1,
    },
  ] satisfies ReadonlyArray<{ name: string; sessions: Session[]; live: number; expired: number }>)(
    "$name",
    ({ sessions, live, expired }) => {
      const got = classifySessions(sessions, now);
      expect(got.live).toHaveLength(live);
      expect(got.expired).toHaveLength(expired);
    },
  );
});

describe("parseFill", () => {
  // Happy path: assert on the whole structured value.
  test("ok", () => {
    expect(parseFill({ sym: "AAPL", qty: "10", px: "190.5" })).toEqual({
      ok: true,
      value: { sym: "AAPL", qty: 10, px: 190.5 },
    });
  });

  // Error paths: pin the specific failure value, not just "it threw".
  test("missing qty", () => {
    expect(parseFill({ sym: "AAPL", px: "190.5" })).toEqual({
      ok: false,
      error: { kind: "missing-field", field: "qty" },
    });
  });

  // A blank string must not coerce to 0 — the bug the explicit guard prevents.
  test("blank qty", () => {
    expect(parseFill({ sym: "AAPL", qty: "  ", px: "190.5" })).toEqual({
      ok: false,
      error: { kind: "bad-number", field: "qty" },
    });
  });

  // A missing symbol must not silently become the string "undefined".
  test("missing sym", () => {
    expect(parseFill({ qty: "10", px: "190.5" })).toEqual({
      ok: false,
      error: { kind: "missing-field", field: "sym" },
    });
  });
});
```

Run with `pnpm vitest run`; a failed `expect` exits non-zero, which is the CI signal. Where a function's contract is a property rather than a fixed table — "classifying never loses a session" — express it with `fast-check` (`fc.assert(fc.property(...))`) rather than enumerating cases.

## Further reading

When a rule above isn't enough to settle a question, these are the canonical sources behind it:

- [Gary Bernhardt — Boundaries](https://www.destroyallsoftware.com/talks/boundaries) — the original Functional Core / Imperative Shell talk; the foundation everything else in this file rests on.
- [Alexis King — Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) — type-driven design as the actionable practice behind "make illegal states unrepresentable." Written in Haskell, but the lessons transfer directly to typed TypeScript.
- [Scott Wlaschin — Designing with types: making illegal states unrepresentable](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/) — where the phrase originates; F#, but the modelling moves are the same ones the section above applies in TypeScript.
- [TypeScript Handbook — Narrowing & Discriminated Unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) — exhaustiveness checking with `never`, the basis for compiler-enforced variant handling.
- [Total TypeScript — branded types and `satisfies`](https://www.totaltypescript.com/) — the idioms behind nominal IDs and checking values against a type without widening.
- [ts-pattern](https://github.com/gvergnaud/ts-pattern) — exhaustive pattern matching for discriminated unions and `Result` types.
- [neverthrow](https://github.com/supermacro/neverthrow) — `Result`/`ResultAsync` for the layers where error composition is the right tool.
- [Zod](https://zod.dev/) — schema validation for turning loose external input into a trusted typed shape once at the boundary.
- [Standard Schema](https://standardschema.dev/) — the `~standard` interface common to Zod, Valibot, and ArkType, so the boundary depends on validation structurally rather than on one library.
- [Valibot](https://valibot.dev/) / [ArkType](https://arktype.io/) — schema validators in the same role as Zod when bundle size (Valibot) or type-level expressiveness (ArkType) is the deciding factor.
- [fast-check](https://fast-check.dev/) — property-based testing; the TypeScript counterpart to Python's Hypothesis.
- [Effect](https://effect.website/) — the heavier alternative when you outgrow the lightweight stack; note that its `Context`/`Layer` system is a DI container, which is why this guidance does not adopt it by default.
