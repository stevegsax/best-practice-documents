# Nushell guidelines

## General Guidelines

- **Target Nushell 0.113.0.** Check it at the boundary (`version | get version`) and fail fast on a mismatch. Nushell makes breaking syntax changes between minor releases; "works on my machine" is a version statement, not a fact. Every concrete syntax in this document is verified against 0.112.2 — re-verify before assuming it holds on a different version.
- Scripts are structured-data programs that happen to live in a shell, not text pipelines. Pass tables, records, and lists through pipelines — never re-parse your own stringified output.
- Correctness and testability are more important than performance.
- Adopt a `functional programming` design: commands are pure transformations of structured data; side effects (I/O, `$env` mutation, external commands) are pushed to the edges.
- Apply the "Functional Core / Imperative Shell" pattern.
- Prefer the pipeline combinators (`each`, `where`, `reduce`, `filter`, `update`, `insert`, `merge`, `reject`, `select`) over `mut` + `for`.
- Prefer Nushell built-ins over spawning external commands. A built-in returns structured data and is portable; an external returns a string you then have to re-parse.
- When an external command is genuinely required, invoke it with an explicit `^` (`^git`, `^docker`) so the dependency on the host environment is visible at the call site.

## Functional Core / Imperative Shell — practice rules

The pattern survives contact with everyday scripting only when these rules are followed in the code itself. Several are contrary to common shell habits; treat them as deliberate departures.

### Module structure

- **One module per file.** `export def` for the public surface; plain `def` stays private. A script's executable entrypoint is `def main []`.
- **Imports go inward, never reverse.** A pure module MUST NOT `use` a module that does I/O, mutates `$env`, or calls externals. If you need to, the boundary is in the wrong place — push the I/O out, not the import in.
- **`use`, not `source`.** `source` splices code into the caller's scope at parse time and is a side effect on the caller; reserve it for the top-level script wiring. Library code is imported with `use` / `export use`.
- **`const` for import paths.** Nushell parses an entire file before evaluating any of it, so a `let` path does not exist yet when `use`/`source` runs. Paths fed to `use`/`source` must be parse-time constants — define them as `const`, derive them from `$nu` (e.g. `$nu.default-config-dir`) or a passed-in root, and never hardcode an absolute path.
- **Prefer the submodule import path for `std`.** `use std/assert`, `use std/log` — the `std/<sub>` form loads only that submodule. `use std assert` (space form) still works in 0.112.2 but pulls more than you asked for; don't use it.
- **No import-time side effects.** Importing a module must not open files, hit the network, or mutate `$env`. Registration/wiring happens when the shell calls an exported command, not as a consequence of `use`.

### Immutability by default

- **`let` is the default; `mut` is a code smell to justify.** Reach for a pipeline (`reduce`, `each`, `generate`) before a `mut` accumulator. A `mut` is acceptable only when the imperative form is genuinely more readable than the fold, and never escapes the command it lives in.
- **`const` for compile-time constants.** Anything known at parse time (paths, lookup tables, version pins) is `const`, signalling it is not data that flows.
- **Records and tables are immutable values.** Produce new values with `update`, `insert`, `merge`, `reject`, `select` — never model a workflow as "mutate this record in place."
- **Closures must not capture `mut` variables to smuggle state out.** A closure that writes to enclosing mutable state is a hidden side effect; return the value instead.

### Dependency injection — by parameter, not by ambient `$env`

- **Pass dependencies as parameters or flags.** Paths, base URLs, timeouts, feature switches: declared parameters with types, wired by the shell at the outermost layer (`main`, the script's argument parsing). Typed signatures make the wiring explicit.
- **The pure core never reads `$env`.** `$env` is mutable global state. The shell reads what it needs from `$env` once, validates it, and passes concrete values down. A core command that reaches into `$env.SOMETHING` is hiding its real inputs.
- **Pure commands take only the values they need, not the whole config record.** Pass `$config.retry_count`, not `$config`. A command that takes the entire settings record is concealing its dependencies.

### Hidden state — none

- **No module-level mutable state.** There is no legitimate module-scoped `mut`.
- **`def --env` is a side effect.** A command that mutates the caller's environment (`def --env`, `cd`, `load-env`, `$env.X = ...`) belongs in the shell, never in the core. Keep the count of such commands small and obvious.
- **Stateful resources live in the shell.** Open file handles, a chosen working directory, an authenticated session: established once at the boundary, with the concrete values passed inward. The core never sees the session — only the token string it needs.

### Type system discipline

- **Annotate every parameter and the pipeline signature.** `def parse-trades [source: path, --strict]: nothing -> table` — both the parameter types and the `input -> output` signature are mandatory, not optional polish.
- **Annotations are parse-time-enforced, not just documentation.** In 0.112.2 a statically-typed mismatch (`parse-trades 42` where `source: path`) is a hard parse error before anything runs. Runtime checking of dynamically-typed values flowing into a typed parameter is gated behind the `enforce-runtime-annotations` experimental option, which is **off** by default — so a wrong type smuggled in through a pipeline can still reach the body. Validate such values explicitly at the boundary; don't assume the signature caught it.
- **Use the precise type, not a vague one.** `path`, `datetime`, `duration`, `filesize`, `cell-path`, `closure`, `binary`, `range` exist for a reason. Use `record<name: string, qty: int>` and `table<sym: string, px: float>` signatures to document the shape of structured data at command boundaries.
- **Avoid `any`.** If a signature must use `any`, leave a comment explaining what the real contract is and why it can't be expressed yet.
- **Doc comments are the contract.** Comment lines immediately above a `def` become its `help` text — write them as the spec a caller reads. In 0.112.2, attach worked examples with the attribute form `@example "description" { 1 | add-one } --result 2` and aliases with `@search-terms "increment"`, placed on the lines directly above the `def`; both surface in `help <command>`.
- **Validate external data into a typed shape at the boundary.** Data from `open`, `http get`, or `from json` is untyped until you check it. "Parse, don't validate": the shell turns it into a known record/table shape once, and the core then trusts the type.

### I/O and externals at the boundary

- **These are shell-only:** `open`, `save`, `http`, `^external`, `input`, `cd`, `rm`, `cp`, `mv`, `mkdir`, `glob`, `ls`, `ps`, `sys`. A pure command takes structured data and returns structured data with none of these in its body.
- **Parse at the edge, compute in the core.** `open data.json | from json | transform-trades` — `transform-trades` receives a parsed value and never knows where it came from. It is then testable with a literal record.
- **`par-each` in the core is fine if its closure is pure.** Parallelism is a property of pure transformations. Concurrency that does I/O (parallel `http get`) is shell concern — quarantine it there.
- **External command output is text until proven otherwise.** Convert (`| from json`, `| lines`, `| parse`) and validate at the boundary; the core never receives a raw external string.

### Side effects: logging, printing, time, randomness

- **`print` is a side effect; it is banned in core logic.** Core commands return values. The shell decides what to render and how.
- **Logging belongs to the shell.** `use std/log` and call `log info`/`log warning`/`log error` at the boundary. The core returns structured results; the shell logs what it judges worth logging about them.
- **Time, randomness, identity, and the system are pseudo-I/O.** `date now`, `random`, `whoami`, `sys host`, `$nu`, `$env` produce different values each run. Accept them as parameters in the core; produce them at the shell. Tests then pass fixed values with no monkeypatching.
- **A `datetime` has no stable default rendering — format it explicitly at the boundary.** Verified in 0.112.2: string interpolation of a datetime yields `Sat, 16 May 2026 12:00:00 +0000 (2 days ago)` (a human string with a relative-time suffix), `into string` yields the ctime-style `Sat May 16 12:00:00 2026`, `to json` emits `+00:00` not `Z`, and `to csv` uses a space instead of `T`. None of these is a contract. Any datetime that leaves the program — JSON/CSV output, a filename, a log line, an external-command argument, an expected value in an assertion — must be rendered with an explicit `format date` strftime string, e.g. `$t | format date "%Y-%m-%dT%H:%M:%SZ"` → `2026-05-16T12:00:00Z`. The core passes `datetime` values around untouched; the shell decides the wire format. Comparison and arithmetic on `datetime` values are safe and need no formatting — only serialization does.

### Configuration and environment

- **Read `$env` only at the boundary, once.** Don't scatter `$env.X?` through the code; the values flow from a single validated config record assembled at startup.
- **Fail fast on missing required config.** A missing required environment variable should `error make` at startup, not surface as a confusing failure ten commands later.
- **Follow XDG for config, cache, and data paths** (`$env.XDG_CONFIG_HOME`, etc., with spec-correct defaults), resolved at the boundary and passed inward as concrete `path` values.

### Error handling

- **`error make` with structure, not bare strings.** Provide `msg`, a `label` with `text` and `span`, and `help` when there is a clear remedy: `error make { msg: "...", label: { text: "...", span: (metadata $arg).span }, help: "..." }`. The span must come from a command parameter or other user value — `(metadata $env)` / `(metadata $nu)` raise "have no metadata" in 0.112.2, so never build a span from a builtin. Omit `label` entirely when there is no user value to point at.
- **Distinguish expected absence from failure.** A lookup that may legitimately find nothing returns `null` (or an empty table); reserve `error make` for genuinely exceptional conditions. The pure core rarely raises.
- **Catch narrowly.** Wrap only the failing operation in `try { ... } catch { |err| ... }`, not an entire pipeline. A `try` around a whole block is "I don't know what fails here" — that is a design problem. Re-raise with added context rather than swallowing.
- **`exit` is shell-only.** Process exit codes are set at the boundary. Core commands return values; they never call `exit`.

### Make illegal states unrepresentable

- **Encode invariants in the record shape.** If two fields can never be set together, model the variants, not two nullable columns side by side.
- **Discriminated unions via a `kind` field + `match`.** Tag each variant with a `kind: string` and branch with `match`. Make the match exhaustive; the final `_ => (error make ...)` arm turns an unhandled variant into a loud failure instead of silent fall-through.
- **Validate once at the boundary, trust thereafter.** The shell turns loose external input into a known typed shape; the core is written against that shape and does not re-check it everywhere.

## Test strategy

The Functional Core / Imperative Shell commitment shapes the tests. The mass of the suite is pure unit tests because the architecture pushes most logic into the core, where testing is a literal-in / literal-out comparison.

- **Pure tests** — functional-core commands. A test calls the command with literal structured data and asserts on the returned structured data. No `open`, no `http`, no `^external`, no fixtures beyond the input values. These are fast; exhaustive case tables are cheap and encouraged.
- **Integration tests** — imperative-shell commands. Real files under `mktemp -d`, real external commands, real (sandboxed) endpoints. Grouped separately from the pure suite. NOT mocks of your own commands.
- **End-to-end tests** — invoke the script through its `main` / CLI boundary with real arguments and assert on observable output and exit code.

**`std/assert` is the test runner.** It ships with 0.112.2 — `use std/assert`, then `assert`, `assert equal`, `assert error`; a case table iterated with `each` gives parametrization. That covers the pure suite with zero install dependencies. A third-party runner (`nutest` via `nupm`, etc.) is an explicit, justified exception — never a default — since it adds an install dependency and a second way to write tests. Don't reach for one to get features `std/assert` plus `each` already provide; if the project already standardised on one, follow it, but don't introduce a second.

Rules that apply across layers:

- **Do not mock your own commands.** If a test needs to stub a collaborator command, either it should be a real integration test, or the logic worth testing is pure and belongs in a unit test of that pure command.
- **Mock only what is genuinely external** and infeasible to run for real — and prefer a real sandboxed resource over a mock.
- **Time, randomness, identity, `$env` are pseudo-I/O.** Pass deterministic values in tests; wire the real source in production.
- **Assert on observable behavior.** The returned record/table, the file written, the exit code — not `to text` rendering, column order, or which private command was called.

Anti-patterns to flag in review:

- Mocking internal collaborator commands.
- A bare `try`/`catch` swallowing errors, or a test that asserts only "it didn't error" — assert the specific `error make` message/label.
- Hardcoded production or home-relative paths in tests — use `mktemp -d`.
- A pure-command test that needs filesystem or network setup — the command isn't pure; fix the command, not the test.
- Re-parsing the command's own stringified output instead of asserting on the structured value it returns.
- `source` used for code reuse in library modules (use `use`); side-effectful imports.
- A test "almost the same as that other test" — promote the variants to a case table iterated with `each`.

### Worked example

A pure core module and its test file, verified against 0.112.2. The test demonstrates: a deterministic pseudo-I/O parameter (`now`) instead of `date now`; a case table iterated with `each`; assertions on the returned structured value rather than its rendering; an error path that pins the message instead of asserting only that something failed; and `std/assert` as the runner with an explicit suite you maintain.

`src/trades.nu` — functional core, no I/O:

```nu
# Partition sessions into live and expired relative to `now`.
#
# `now` is passed in (pseudo-I/O) so the command stays pure and the
# tests are deterministic — no `date now` reaching into the clock.
export def classify-sessions [
    now: datetime
]: table<id: string, expires: datetime> -> record<live: table, expired: table> {
    let sessions = $in
    {
        live:    ($sessions | where expires > $now)
        expired: ($sessions | where expires <= $now)
    }
}

# Parse a raw fill into a typed record, erroring on a missing quantity.
export def parse-fill [
    raw: record
]: nothing -> record<sym: string, qty: int, px: float> {
    if ($raw.qty? | is-empty) {
        error make {
            msg: "fill is missing qty"
            label: { text: "offending fill", span: (metadata $raw).span }
            help: "every fill needs an integer qty"
        }
    }
    { sym: $raw.sym, qty: ($raw.qty | into int), px: ($raw.px | into float) }
}
```

`tests/test_trades.nu`:

```nu
use std/assert
use ../src/trades.nu *

# A pure command + a deterministic `now` + a case table iterated with `each`,
# asserting on the returned structured value (not its rendering).
def "test classify-sessions" [] {
    let now = 2026-05-16T12:00:00Z
    let cases = [
        [name          sessions                                       live  expired];
        ["all live"     [{ id: "a", expires: 2026-05-17T00:00:00Z }]   1     0      ]
        ["all expired"  [{ id: "b", expires: 2026-05-15T00:00:00Z }]   0     1      ]
        ["mixed"        [{ id: "c", expires: 2026-05-17T00:00:00Z }
                         { id: "d", expires: 2026-05-15T00:00:00Z }]   1     1      ]
    ]
    $cases | each {|c|
        let got = ($c.sessions | classify-sessions $now)
        assert equal ($got.live | length) $c.live
        assert equal ($got.expired | length) $c.expired
    }
}

# Happy path: assert on the whole structured value.
def "test parse-fill ok" [] {
    assert equal (parse-fill { sym: "AAPL", qty: "10", px: "190.5" }) { sym: "AAPL", qty: 10, px: 190.5 }
}

# Error path: pin the specific failure, not just "it errored somehow".
def "test parse-fill missing qty" [] {
    let err = (try { parse-fill { sym: "AAPL", px: "190.5" }; null } catch {|e| $e })
    assert ($err != null) "expected parse-fill to raise"
    assert ($err.msg | str contains "missing qty")
}

# std/assert is the runner: discovery is an explicit suite you maintain.
let suite = [
    { name: "classify-sessions",      run: { test classify-sessions } }
    { name: "parse-fill ok",          run: { test parse-fill ok } }
    { name: "parse-fill missing qty", run: { test parse-fill missing qty } }
]
$suite | each {|t| do $t.run; print $"ok: ($t.name)" }
print "all tests passed"
```

Run with `nu tests/test_trades.nu` from the project root; a failed `assert` aborts with a non-zero exit, which is the CI signal.

## Test safety

Tests MUST NOT write to real config, data, or cache paths, and MUST NOT run destructive external commands against real systems.

- Confine all filesystem writes to a `mktemp -d` directory; never `save`/`rm`/`mv` outside it.
- Override XDG paths in test setup with `with-env` (`XDG_CONFIG_HOME`, `XDG_DATA_HOME`, `XDG_CACHE_HOME` pointing into the temp dir); never let a test touch `~/.config`, `~/.local/share`, `~/.cache`.
- No network in the pure suite. Network only in explicitly marked integration tests, against a sandbox endpoint.
- An autouse/setup step should refuse to run a destructive integration suite unless the target endpoint or database name carries a test marker (`test`, `_dev`, `localhost`, `127.0.0.1`).
- Type-annotate test helper commands too; they are code.

## Further reading

When a rule above isn't enough to settle a question, these are the canonical sources behind it:

- [Gary Bernhardt — Boundaries](https://www.destroyallsoftware.com/talks/boundaries) — the original Functional Core / Imperative Shell talk; the foundation everything else here rests on.
- [Nushell Book — Thinking in Nu](https://www.nushell.sh/book/thinking_in_nu.html) — why Nushell is a structured-data language, parse-time vs run-time, and why "source isn't import."
- [Nushell Book — Custom Commands & type signatures](https://www.nushell.sh/book/custom_commands.html) — parameter types, input/output signatures, doc comments as help.
- [Nushell Book — Creating Errors](https://www.nushell.sh/book/creating_errors.html) — `error make`, labels, and spans.
- [Nushell — `format date`](https://www.nushell.sh/commands/docs/format_date.html) — the strftime formatter; the only reliable way to serialize a `datetime` to a stable string.
- [Nushell Standard Library](https://www.nushell.sh/book/standard_library.html) — `std/assert`, `std/log`, and the testing utilities.
- [Alexis King — Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) — the discipline behind "validate external data into a typed shape once at the boundary." Haskell, but the lesson transfers directly.
