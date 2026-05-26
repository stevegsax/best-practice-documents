# Python guidelines

## General Guidelines

- Use `uv` to manage python dependencies and modules
- Correctness and testability are more important than performance
- Where possible, adopt a `functional programming` design philosophy where functions are idempotent and side effects are handled separately
- Apply the "Functional Core / Imperative Shell" pattern
- Prefer `map`, `reduce`, and `filter` for list processing
- Prefer `itertools` iteration and looping instead of python comprehensions
- Prefer `functools` for functional programming and higher order functions

## Scripts and helpers

- **Run scripts with `uv run`, not a bare `python`.** It resolves the interpreter and dependencies in one step — no manually activated virtualenv to forget about.
- **Declare dependencies inline with PEP 723** for standalone helper scripts (e.g., the executable part of a skill). Embed the dependency list in the script itself rather than a sidecar `requirements.txt` or an assumed ambient environment; `uv run` reads the metadata and builds an ephemeral environment per run, so the script stays self-contained and reproducible wherever it lands:

  ```python
  # /// script
  # requires-python = ">=3.12"
  # dependencies = ["httpx", "rich"]
  # ///
  ```

## Functional Core / Imperative Shell — practice rules

The pattern survives contact with everyday development only when these rules are followed in the code itself. Most of them are contrary to mainstream Python defaults; treat them as deliberate departures.

### Module structure

- **Imports go inward, never reverse.** Pure modules MUST NOT import from modules that do I/O, manage state, or call framework APIs. If you need to, the boundary is in the wrong place — push the I/O out, not the import in.
- **Side-effect imports are an anti-pattern**, even when they look intentional. Don't register routes / steps / hooks / plugins via import-time side effects. Use the framework's own discovery mechanism (entry points, conftest hierarchy, plugin specs, decorators that the framework collects). Side-effect registration depends on the importer running at the right time and breaks silently across framework versions.

### Immutability by default

- **`@dataclass(frozen=True)` for value types.** Default to immutable; mutate via `dataclasses.replace`. The ~2.4× instantiation cost is invisible against any I/O and pays for itself the first time something doesn't mutate when you expected it not to.
- **`Mapping`, `Sequence`, `Iterable` over `dict`, `list` in function signatures.** Signals a read-only contract and lets callers pass any compatible type.
- **Return `tuple`, not `list`, from pure functions.** Lists imply "callers may mutate me"; tuples don't.

### Dependency injection — by parameter, not by container

- **Pass dependencies as parameters; do NOT use a DI container.** No `dependency-injector`, no service locators, no `kink`. The shell wires concrete dependencies at the outermost boundary (framework hooks like FastAPI `Depends`, CLI argparse, etc.) and passes them down through the call chain. Typed function signatures make the wiring explicit.
- **No module-level singletons for I/O clients.** DB engines, HTTP clients, object-storage clients live in the shell, are created at startup, then passed explicitly. Tests substitute test fixtures the same way production wires real clients.
- **Pure functions take only the values they need, not container objects.** A pure function that takes a full `Settings` is hiding its real dependencies — pass `settings.session_secret_lifetime_seconds` instead.

### Hidden state — none

- **No module-level mutable state.** Globals are not load-bearing for any code path you care about. If something seems to need it, you've conflated a value (config) with a service (something stateful).
- **No `functools.lru_cache` on impure functions.** Memoization assumes referential transparency. Caching results of I/O calls is a different concern handled at the shell, deliberately.
- **Stateful resources (pools, caches, sessions) live in the shell.** Created once at startup, passed explicitly. The pure core never sees a pool — only the values it needs from one.

### Type system discipline

- `from __future__ import annotations` in every file.
- mypy strict mode (`strict = true`, `disallow_untyped_defs = true`). Treat new errors as build breaks, not warnings.
- `Protocol` over ABC for duck-typed contracts — structural typing matches Python's nature.
- `NewType` for ID types that shouldn't mix: `UserId = NewType("UserId", uuid.UUID)`. Typos that "happen to compile" are bugs in waiting.
- `Final` for module-level constants. `Literal` for closed sets. `TypedDict` with `NotRequired` for structured dicts at boundaries (e.g., webhook payloads, API JSON).
- Avoid `Any`. If you must reach for it, leave a comment explaining why.

### Async at the boundary

- **`async def` implies I/O — it lives in the shell.** Pure-core functions are sync. This is contrary to a lot of Python codebases that make everything async; do not follow that lead.
- **Don't propagate `async` farther than needed.** Once a function is async, every caller has to be too. Quarantine it to the shell layer; pure transformations stay sync and remain trivially testable.

### Side effects: logging, metrics, time, randomness

- **Logging is a side effect.** It belongs in the shell. Pure functions return values; the shell decides what to log about them.
- **Time, randomness, UUIDs, hostname, environ are pseudo-I/O.** Accept them as parameters in the core; produce them at the shell. Tests pass deterministic values without monkeypatching.
- **No `print()` in business logic** — ever. Use logging at the shell or return values the boundary can format.

### Configuration

- **One `Settings` class** (e.g., `pydantic-settings`), loaded once at app startup, validated at boundary.
- **Read env vars only inside `Settings`.** Don't sprinkle `os.environ.get(...)` through the code; the values flow from the single loaded Settings object.
- **In production, fail fast on missing required config.** A missing `DATABASE_URL` should crash startup, not surface as a 500 ten minutes later.

### Error handling

- **Domain-specific exception types**, not bare `Exception`. The type IS the error category; the message carries the human detail.
- **Pure-core functions rarely raise.** Encode failures in return types where it makes sense (`parse() -> ParseResult | ParseError`); reserve exceptions for genuinely exceptional conditions.
- **Catch narrow.** `except SpecificError:` only. `except Exception:` signals "I don't know what can go wrong here" — that's a design problem, not a coding problem.
- **Result / Either types (`returns` library) are a sometimes-tool.** Reach for them when error composition matters (a pipeline of fallible steps where each may fail differently); use exceptions for one-off failures. Don't mix styles within a module — pick one per layer.

### Make illegal states unrepresentable

- **Encode invariants in types.** If two fields can never be set together, model the union, not two `Optional` fields.
- **Discriminated unions** via a `Literal["kind"]` field on a dataclass + `match` at the boundary. Adding a new variant becomes a compiler-enforced exhaustiveness check.
- **`NewType` for IDs everywhere.** `UserId`, `DocumentId`, `SessionId` should be distinct types even if all wrap UUIDs.

## Test strategy

The Functional Core / Imperative Shell commitment shapes every test. The pyramid has three layers — its mass is in pure unit tests because the architecture pushes most logic into the core, where testing is cheap.

- **Pure tests** — functional-core code. Conventionally under `tests/unit/`. No I/O, no fixtures except `parametrize` and `hypothesis` strategies. Should run in microseconds. Exhaustive parametrization and property-based testing are encouraged precisely because they're cheap here.
- **Integration tests** — imperative-shell code. Conventionally under `tests/integration/`, marked `@pytest.mark.integration`. Use real dependencies: ephemeral Postgres via `pytest-postgresql`, real cloud-service buckets/queues isolated by per-run prefix, sandbox SMTP. NOT mocks of your own code.
- **End-to-end / BDD tests** — full-stack behavioral coverage. Step definitions or test bodies delegate to HTTP/CLI boundaries, not service-layer internals.

Rules that apply across layers:

- **Do not mock your own collaborators.** Mocking your service layer in a route test almost always means either (a) it should be a real integration test, or (b) the logic worth testing is pure and belongs in a unit test of that pure function.
- **Mock only what's genuinely external.** Third-party services you don't control: `respx` for outbound HTTP, `moto` for AWS APIs as a last resort when a real test resource is infeasible.
- **Time, randomness, UUIDs, hostname, environ, etc. are pseudo-I/O.** Accept them as parameters in the core; supply them at the shell. Tests pass deterministic values; production wires the real source.
- **Assert on observable behavior, not implementation.** Row written, file present, HTTP status and response body. NOT internal field order, `__repr__` output, or which private method was called with what arguments.

Anti-patterns to flag in review:

- Mocking internal collaborators.
- `except Exception:` or `pytest.raises(Exception)` — pin the specific type and a message substring.
- Hardcoded production paths (use `tmp_path`).
- A pure-function unit test that needs more than `parametrize`-worth of setup — the function isn't pure; fix the function, not the test.
- Side-effect imports for plugin/step/route registration — use the framework's own discovery mechanism (conftest hierarchy, entry points, plugin specs). Side-effect registration is fragile and silently breaks across framework versions.
- A test that's "almost the same as that other test" — promote shared setup to a fixture or `parametrize`.
- A refactor that requires editing tests — either observable behavior changed (then it's not a refactor) or the test was coupled to implementation (fix the coupling, don't preserve it).

## Test safety

Tests MUST NOT write to production databases, config paths, or cache directories.

- Use the project's test fixtures (e.g. `test_database`, `cli_runner`) rather than constructing `CliRunner()` or database connections directly.
- Never hardcode XDG production paths — `~/.local/share/`, `~/.config/`, `~/.cache/`. Let fixtures override these via env vars or temp dirs.
- Before writing new tests, check `conftest.py` for existing safety fixtures and isolation helpers.
- An autouse fixture should refuse to run the suite if the active `DATABASE_URL` doesn't contain a test marker (`test`, `_dev`, `localhost`, `127.0.0.1`). Same principle for any other backing service that has a production endpoint reachable from a dev machine.
- Include type hints for parameters and return values.

## Further reading

When a rule above isn't enough to settle a question, these are the canonical sources behind it:

- [Gary Bernhardt — Boundaries](https://www.destroyallsoftware.com/talks/boundaries) — the original Functional Core / Imperative Shell talk; the foundation everything else in this file rests on.
- [Alexis King — Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) — type-driven design as the actionable practice behind "make illegal states unrepresentable." Written in Haskell but the lessons transfer directly to typed Python.
- [Hillel Wayne — Constructive vs Predicative Data](https://www.hillelwayne.com/post/constructive/) — useful counterpoint on where pure constructive typing reaches ergonomic limits and predicate-based validation is the better tool.
- [dry-python/returns — Railway-Oriented Programming](https://returns.readthedocs.io/en/latest/pages/railway.html) — reference for when `Result` / `Either` is the right tool for a specific layer (and the idioms that make it readable in Python).
- [PEP 655 — `Required` / `NotRequired` for TypedDict](https://peps.python.org/pep-0655/) — the language feature behind precise typing of structured payloads at API/webhook boundaries.
