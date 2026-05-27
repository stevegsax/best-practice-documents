# Python guidelines

## General Guidelines

- Use `uv` to manage Python dependencies, virtual environments, and tools. Commit `uv.lock` and pin the interpreter (`.python-version`, and `requires-python` in `pyproject.toml`) so a build is reproducible across machines and CI rather than dependent on an ambient environment — table stakes for a regulated trading shop.
- Use `ruff` to lint and format, run in CI. Many rules below are mechanically enforceable — bare `except`, `print` in library code, mutable default arguments, unused imports and `# type: ignore`s — so let the linter enforce them instead of relying on review.
- Target a current Python (3.13+, ideally 3.14). Several rules here assume modern stdlib and typing: `asyncio.TaskGroup`, `typing.assert_never`, `StrEnum`, PEP 695 generics, and PEP 649 deferred annotations.
- Correctness and testability are more important than performance.
- Where possible, adopt a `functional programming` design philosophy where functions are **pure** — deterministic and free of side effects — and side effects are handled separately.
- Apply the "Functional Core / Imperative Shell" pattern.
- Prefer comprehensions and generator expressions for transformation and filtering — they are expression-based and don't mutate, which fits the functional core. Use a generator expression when the result is consumed lazily or could be large.
- Use `map`/`filter` only when applying an existing named function (`map(parse_row, rows)`), where they read more cleanly than a comprehension. Avoid `functools.reduce` — reach for `sum`/`any`/`all`/`min`/`max`/`math.prod`, or an explicit loop when the accumulation is non-trivial.
- Reach for `itertools` for lazy, streaming, or combinatorial pipelines (`chain`, `groupby`, `islice`, `accumulate`) — not as a general substitute for comprehensions. Use `functools` deliberately: `partial` and `singledispatch` for composition, `wraps` for decorators; keep `lru_cache`/`cache` to the shell (see *Hidden state*).

## Scripts and helpers

- **Run scripts with `uv run`, not a bare `python`.** It resolves the interpreter and dependencies in one step — no manually activated virtualenv to forget about.
- **Declare dependencies inline with PEP 723** for standalone helper scripts (e.g., the executable part of a skill). Embed the dependency list in the script itself rather than a sidecar `requirements.txt` or an assumed ambient environment; `uv run` reads the metadata and builds an ephemeral environment per run, so the script stays self-contained and reproducible wherever it lands:

  ```python
  # /// script
  # requires-python = ">=3.13"
  # dependencies = ["httpx", "rich"]
  # ///
  ```

- **Lock standalone scripts too.** `uv lock --script helper.py` writes a companion lockfile beside the script, so even a one-file helper resolves to the same dependency versions on every run.

## Functional Core / Imperative Shell — practice rules

The pattern survives contact with everyday development only when these rules are followed in the code itself. Most of them are contrary to mainstream Python defaults; treat them as deliberate departures.

### Module structure

- **Imports go inward, never reverse.** Pure modules MUST NOT import from modules that do I/O, manage state, or call framework APIs. If you need to, the boundary is in the wrong place — push the I/O out, not the import in.
- **Side-effect imports are an anti-pattern**, even when they look intentional. Don't register routes / steps / hooks / plugins via import-time side effects. Use the framework's own discovery mechanism (entry points, conftest hierarchy, plugin specs, decorators that the framework collects). Side-effect registration depends on the importer running at the right time and breaks silently across framework versions.
- **Type-only imports go under `if TYPE_CHECKING:`.** Imports needed solely for annotations — especially of shell or I/O types — must not run at import time. Guard them with `from typing import TYPE_CHECKING` so a pure module never pulls in a heavy or effectful dependency just to name a type.
- **Declare the public surface with `__all__`; never `from module import *`.** A star-import defeats "find references," silently shadows names, and couples the importer to a module's incidental contents.

### Immutability by default

- **`@dataclass(frozen=True, slots=True)` for value types.** Default to immutable; produce changes with `dataclasses.replace`. Frozen routes `__init__` through `object.__setattr__`, so instantiation runs ~2.4× a plain dataclass — invisible against any I/O — and `slots=True` claws most of that back (less memory, faster attribute access) while `kw_only=True` stops fields being swapped positionally in wide value types.
- **Frozen is shallow — freeze the fields too.** `frozen=True` blocks rebinding a field, not mutation of the value it points at: a frozen dataclass with a `list` field still allows `obj.items.append(...)`. Use `tuple`, `frozenset`, and `Mapping`/`types.MappingProxyType` for collection fields so the whole value is actually immutable.
- **Never use a mutable default argument.** `def f(items: list = []):` shares one list across every call — the canonical Python footgun and a direct violation of this section. Use `None` and create inside, or an immutable default. `ruff` flags it (B006).
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
- **Module top level runs on import — so it must not do I/O.** A module-level `httpx.get(...)`, file read, or client construction executes the moment the module is imported (including by the test collector), coupling import to side effects and import order. Build such resources once in the shell and pass them inward.

### Type system discipline

- **Drop `from __future__ import annotations`.** On Python 3.14+ (PEP 649) annotations are evaluated lazily by default, so the import is redundant — and because it *stringizes* annotations it breaks runtime-annotation consumers like pydantic and FastAPI, which this guide mandates. If you must still support 3.12–3.13, add it only in modules with forward-reference cycles that aren't introspected at runtime.
- **mypy strict is a build gate, not a warning.** Enable `strict = true` plus, explicitly, `disallow_untyped_defs`, `warn_return_any`, `warn_unused_ignores`, `warn_redundant_casts`, and `no_implicit_reexport`; run it in CI and treat new errors as build breaks. `pyright --strict` is an equally valid gate. The fast checkers `ty` (Astral) and `pyrefly` (Meta) are worth running in-editor for speed, but keep mypy or pyright as the authoritative gate until they reach parity.
- **Money is `Decimal` or integer minor units — never `float`.** Binary floating point cannot represent `0.1`, and `0.1 + 0.2 != 0.3`. For prices, quantities, and notionals use `decimal.Decimal` (constructed from a `str`, never from a `float`) or integer minor units, with an explicit rounding mode set at the boundary. A `float` price is a representable-but-wrong state — exactly what this discipline exists to prevent.
- **Serialize deterministically at the egress boundary.** Stable key order, `Decimal` rendered as a string (never coerced back to `float`), and timezone-aware `datetime` only — never naive. Audit and reproducibility depend on byte-stable output, not just validated input.
- **`Protocol` over ABC for duck-typed contracts** — structural typing matches Python's nature, and a narrow `Protocol` is the dependency-injection seam the core depends on: the shell passes anything that satisfies it.
- **`NewType` distinguishes IDs but does not validate them.** `UserId = NewType("UserId", uuid.UUID)` stops a `DocumentId` being passed where a `UserId` belongs, but `UserId(x)` runs no check — it is a static-only label. When a value needs validating (a positive price, a well-formed symbol), parse it into a frozen single-field dataclass with `__post_init__` validation, or a pydantic `Annotated[...]` type, so the type means "checked," not merely "named."
- **Prefer `object` to `Any` when you mean "any type, but I'll narrow it."** `object` forces an `isinstance`/`TypeGuard` check before use; `Any` switches the checker off and is contagious. For external data, don't reach for `Any` at all — parse it (pydantic / `TypeAdapter`) into a precise type at the boundary. If you must use `Any`, leave a comment; and never write a bare `# type: ignore` — use `# type: ignore[code]` so `warn_unused_ignores` retires it when it's no longer needed.
- **Avoid `typing.cast`** — it's an unchecked assertion (the `as`-equivalent) and is how "impossible" runtime errors happen. Narrow with `isinstance`/`assert`/`TypeGuard` instead; reserve `cast` for the validated boundary where you've already proven the shape.
- **`Final` for constants; closed sets are `Literal` or `Enum`.** `Literal["open", "closed", "settled"]` is the lightweight choice for string tags, especially discriminators; reach for `enum.Enum`/`StrEnum` when you want a named runtime type with iteration, methods, or a value that travels through the system — unlike TypeScript, a Python `Enum` carries no runtime penalty worth avoiding. `TypedDict` with `NotRequired` types structured dicts at boundaries (webhook payloads, API JSON).
- **Modern typing pays its way.** Use PEP 695 syntax (`type Vector = tuple[float, ...]`, `def first[T](xs: Sequence[T]) -> T`) over explicit `TypeVar`; put `@override` (PEP 698) on every method that overrides one, so a renamed base method becomes a type error; and use `typing.assert_never` to make `match` exhaustiveness compiler-checked (see *Make illegal states unrepresentable*).

### Async at the boundary

- **`async def` implies I/O — it lives in the shell.** Pure-core functions are sync. This is contrary to a lot of Python codebases that make everything async; do not follow that lead.
- **Don't propagate `async` farther than needed.** Once a function is async, every caller has to be too. Quarantine it to the shell layer; pure transformations stay sync and remain trivially testable.
- **Concurrent I/O uses structured concurrency.** Fan out with `asyncio.TaskGroup`, not a bare `asyncio.create_task` whose result and exceptions can be silently dropped (the "floating task" bug), and bound it with `asyncio.timeout`. When several tasks fail, the group raises an `ExceptionGroup`; handle it with `except*`. Thread cancellation through rather than firing and forgetting.

### Side effects: logging, metrics, time, randomness

- **Logging is a side effect.** It belongs in the shell. Pure functions return values; the shell decides what to log about them.
- **Time, randomness, UUIDs, hostname, environ are pseudo-I/O.** Accept them as parameters in the core; produce them at the shell. Tests pass deterministic values without monkeypatching.
- **No `print()` in business logic** — ever. Use logging at the shell or return values the boundary can format.

### Configuration

- **One `Settings` class** (e.g., `pydantic-settings`), loaded once at app startup, validated at boundary.
- **Read env vars only inside `Settings`.** Don't sprinkle `os.environ.get(...)` through the code; the values flow from the single loaded Settings object.
- **In production, fail fast on missing required config.** A missing `DATABASE_URL` should crash startup, not surface as a 500 ten minutes later.

### Error handling

- **Domain-specific exception types**, not bare `Exception`. The type IS the error category; the message carries the human detail. Root them at one base app exception so a caller can catch broadly only where it genuinely must, and carry structured data as attributes rather than interpolating it into the message string.
- **Chain with `raise NewError(...) from exc`.** Preserve the originating error explicitly when re-raising across a boundary; use `raise ... from None` only when you deliberately mean to drop the cause. This is the context that turns a production traceback from a guess into a diagnosis.
- **Pure-core functions rarely raise — and Python won't tell you what they do.** A signature `def settle(...) -> Trade` says nothing about what it raises (Python has no checked exceptions), so for the core, encode *expected* failures in the return type (`parse(...) -> Fill | ParseError`) where it makes the failure modes part of the contract. Raising stays the correct default for the shell, for I/O, and for genuinely exceptional conditions.
- **Catch narrow, and never bare.** `except SpecificError:` only. `except Exception:` signals "I don't know what can go wrong here" — a design problem, not a coding one — and a *bare* `except:` (or `except BaseException`) is worse: it swallows `KeyboardInterrupt` and `SystemExit`, so Ctrl-C and shutdown stop working. For a deliberate, specific ignore, use `contextlib.suppress(FileNotFoundError)`, which states the intent.
- **Log exceptions with the traceback.** At the shell, `logger.exception(...)` (or `exc_info=True`) records the traceback and the `__cause__` chain you built above; `logger.error(str(e))` throws both away.
- **Result / Either types (`returns` library) are a sometimes-tool.** Reach for them when error composition matters (a pipeline of fallible steps where each may fail differently) and branch with `match`; use exceptions for one-off failures. Don't mix styles within a module — pick one per layer.

### Make illegal states unrepresentable

- **Encode invariants in types.** If two fields can never be set together, model the union of valid states, not two `Optional` fields that permit the illegal combination.
- **Replace boolean/optional soup with a state union.** Don't model an order as `filled: bool`, `rejected: bool`, `fill_price: Decimal | None` — that makes the impossible combinations (filled *and* rejected; a `fill_price` with no fill) as constructible as the valid ones. Model the legal states as a union of frozen dataclasses with a `Literal` discriminator, and branch with `match` + `assert_never` so adding a variant becomes a type error at every site that handled the old set:

  ```python
  @dataclass(frozen=True, slots=True)
  class Pending:
      kind: Literal["pending"] = "pending"

  @dataclass(frozen=True, slots=True)
  class Filled:
      fill_price: Decimal
      filled_at: datetime
      kind: Literal["filled"] = "filled"

  @dataclass(frozen=True, slots=True)
  class Rejected:
      reason: str
      kind: Literal["rejected"] = "rejected"

  type Order = Pending | Filled | Rejected

  def settled_value(order: Order) -> Decimal:
      match order:
          case Filled(fill_price=price):
              return price
          case Pending() | Rejected():
              return Decimal("0")
          case _ as unreachable:
              assert_never(unreachable)
  ```

- **`NewType` for IDs everywhere** — `UserId`, `DocumentId`, `SessionId` distinct even if all wrap `uuid.UUID` — and validate loose external input into the precise typed shape once at the boundary (pydantic, or a parsing function), so the core trusts the type and doesn't re-check it everywhere.
- **Require "at least one" in the signature.** Python has no clean non-empty-list type; the idiomatic way to demand ≥1 element is a `def vwap(first: Fill, *rest: Fill) -> Decimal` signature, which a caller cannot satisfy with an empty sequence.

## Test strategy

The Functional Core / Imperative Shell commitment shapes every test. The pyramid has three layers — its mass is in pure unit tests because the architecture pushes most logic into the core, where testing is cheap.

- **Pure tests** — functional-core code. Conventionally under `tests/unit/`. No I/O, no fixtures except `parametrize` and `hypothesis` strategies. Should run in microseconds. Exhaustive parametrization and property-based testing are encouraged precisely because they're cheap here. For monetary and parsing logic, lean on `hypothesis` for invariants — round-trips (`parse(render(x)) == x`) and conservation laws (allocation splits sum to the total) — rather than a handful of hand-picked examples.
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

### Worked example

A pure core module and its pytest test. It shows failures encoded as a `Fill | ParseError` return rather than an exception; `Decimal` for money, built from `str`; an injected `now` (pseudo-I/O) so the clock never reaches into the core; a `parametrize` case table; and assertions that pin the exact returned value — including the error — rather than checking only that something raised. A thin shell (not shown) reads the file or HTTP body and hands each raw row to `parse_fill`; the tests exercise the pure core directly.

`trades.py` — functional core, no I/O:

```python
from collections.abc import Mapping
from dataclasses import dataclass
from datetime import datetime, timedelta
from decimal import Decimal, InvalidOperation
from typing import Literal


@dataclass(frozen=True, slots=True)
class Fill:
    symbol: str
    qty: int
    price: Decimal  # money is Decimal, never float


@dataclass(frozen=True, slots=True)
class ParseError:
    field: Literal["symbol", "qty", "price"]
    reason: Literal["missing", "not-a-number"]


def _to_int(value: object) -> int | None:
    if isinstance(value, bool):  # bool is an int subclass — exclude it
        return None
    if isinstance(value, int):
        return value
    if isinstance(value, str) and value.strip():
        try:
            return int(value)
        except ValueError:
            return None
    return None


def _to_decimal(value: object) -> Decimal | None:
    if isinstance(value, Decimal):
        return value
    if isinstance(value, str) and value.strip():  # build from str, never float
        try:
            return Decimal(value)
        except InvalidOperation:
            return None
    return None


# Failures are values, not exceptions, in the core: the return type names them.
def parse_fill(raw: Mapping[str, object]) -> Fill | ParseError:
    symbol = raw.get("symbol")
    if not isinstance(symbol, str) or not symbol.strip():
        return ParseError("symbol", "missing")
    if "qty" not in raw:
        return ParseError("qty", "missing")
    qty = _to_int(raw["qty"])
    if qty is None:
        return ParseError("qty", "not-a-number")
    if "price" not in raw:
        return ParseError("price", "missing")
    price = _to_decimal(raw["price"])
    if price is None:
        return ParseError("price", "not-a-number")
    return Fill(symbol, qty, price)


# `now` is injected (pseudo-I/O) so the core stays pure and deterministic.
def is_stale(fill_time: datetime, now: datetime, *, max_age: timedelta) -> bool:
    return now - fill_time > max_age
```

`tests/test_trades.py`:

```python
from datetime import datetime, timedelta
from decimal import Decimal

import pytest

from trades import Fill, ParseError, is_stale, parse_fill

NOW = datetime(2026, 5, 16, 12, 0, 0)


@pytest.mark.parametrize(
    ("raw", "expected"),
    [
        ({"symbol": "AAPL", "qty": "10", "price": "190.50"}, Fill("AAPL", 10, Decimal("190.50"))),
        ({"symbol": "AAPL", "price": "190.50"}, ParseError("qty", "missing")),
        ({"symbol": "AAPL", "qty": "  ", "price": "190.50"}, ParseError("qty", "not-a-number")),
        ({"qty": "10", "price": "190.50"}, ParseError("symbol", "missing")),
        ({"symbol": "AAPL", "qty": "10", "price": "oops"}, ParseError("price", "not-a-number")),
    ],
)
def test_parse_fill(raw: dict[str, object], expected: Fill | ParseError) -> None:
    assert parse_fill(raw) == expected


@pytest.mark.parametrize(
    ("age", "stale"),
    [(timedelta(minutes=1), False), (timedelta(hours=2), True)],
)
def test_is_stale(age: timedelta, stale: bool) -> None:
    assert is_stale(NOW - age, NOW, max_age=timedelta(hours=1)) == stale
```

Run with `uv run pytest`; `uv run mypy --strict .` and `uv run ruff check` pass on both files. Where a function's contract is a property rather than a fixed table — "a parsed fill round-trips through serialization" — express it with `hypothesis` instead of enumerating cases.

## Further reading

When a rule above isn't enough to settle a question, these are the canonical sources behind it:

- [Gary Bernhardt — Boundaries](https://www.destroyallsoftware.com/talks/boundaries) — the original Functional Core / Imperative Shell talk; the foundation everything else in this file rests on.
- [Alexis King — Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) — type-driven design as the actionable practice behind "make illegal states unrepresentable." Written in Haskell but the lessons transfer directly to typed Python.
- [Scott Wlaschin — Designing with types: making illegal states unrepresentable](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/) — where the phrase originates; F#, but the modelling moves transfer directly to dataclass unions.
- [Hillel Wayne — Constructive vs Predicative Data](https://www.hillelwayne.com/post/constructive/) — useful counterpoint on where pure constructive typing reaches ergonomic limits and predicate-based validation is the better tool.
- [dry-python/returns — Railway-Oriented Programming](https://returns.readthedocs.io/en/latest/pages/railway.html) — reference for when `Result` / `Either` is the right tool for a specific layer (and the idioms that make it readable in Python).
- [PEP 655 — `Required` / `NotRequired` for TypedDict](https://peps.python.org/pep-0655/) — the language feature behind precise typing of structured payloads at API/webhook boundaries.
- [PEP 649 — Deferred evaluation of annotations](https://peps.python.org/pep-0649/) — why `from __future__ import annotations` is no longer the right default on modern Python.
- [pydantic](https://docs.pydantic.dev/) — parsing loose external input into validated, precise types once at the boundary; the practical engine behind "parse, don't validate" in Python.
