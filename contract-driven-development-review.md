# Adversarial review: contract-driven development & static analysis

> Scope: an adversarial pass over the contract-driven development guidance in this repo — primarily `python-guidelines.md`, with `make_spec.md`, `make_todo.md`, `standard_features.md`, and the `CLAUDE.md`/`AGENTS.md`/`GEMINI.md` agent files. Lens: **maximize the type system and contract features, maximize what static analysis can catch, and minimize day-to-day friction.** Audience: a new engineer with minimal experience, so terms are defined and the "why" is spelled out. External best-practice sources are cited at the end.

## TL;DR for the impatient

The guidelines are strong on **types-as-contracts** (frozen dataclasses, `Protocol`, `NewType`, `Literal`/`Enum` unions, "make illegal states unrepresentable", errors-as-return-values). That is the hard half and it is done well.

But the document is called *contract-driven* and it is missing the other half:

1. **There is no Design-by-Contract layer.** Preconditions, postconditions, and invariants — the actual definition of "contract" in software engineering — are never named. The `assert` statement is never mentioned (nor its `python -O` footgun). `icontract`, `deal`, and `CrossHair` do not appear at all.
2. **The static-analysis gates are named but never wired.** The doc says "let the linter enforce it," then ships a ruff selection that omits the rules which would enforce its own prose rules (blind-except, naive datetime, boolean-trap, async-at-boundary, complexity). mypy is declared "a build gate" but no config block, no `enable_error_code`, no `warn_unreachable`, no pre-commit, no CI command.
3. **The reference implementation contradicts the guideline.** `sax-llm` (the repo these docs are meant to govern) has **no mypy, no pyright, no hypothesis, no contract library**, `requires-python = ">=3.12"` (the doc says 3.13+/3.14), a partial ruff rule set, and a **25% coverage floor** while `make_todo.md` demands ">90%". A guideline whose flagship project ignores it teaches a new engineer that the rules are optional.

Fixing #2 and #3 is mostly mechanical and high-leverage. Fixing #1 is the conceptual gap that the word "contract-driven" promises and the doc does not deliver. The rest of this document is the detail.

---

## Part A — Where the guidelines are wrong (or contradict themselves / reality)

These are defects, not matters of taste. A new engineer following the docs literally will hit each of them.

### A1. The agent files point to a directory structure that does not exist

`CLAUDE.md`, `AGENTS.md`, and `GEMINI.md` all describe the repo as organized into `engineering/`, `software-development-process/`, `product-management/`, `state-machines/`, and `specifications/`, and tell the reader the Python standards live at `engineering/best-practices-python.md`, `engineering/python-project-layout.md`, and `engineering/evaluation-framework-python.md`.

**None of those paths exist.** The repo is flat: `python-guidelines.md`, `typescript-guidelines.md`, `make_spec.md`, etc., with the old files archived under `obsolete/`. A new engineer (or an AI agent — these files are explicitly *for* agents) will be sent to phantom files on day one. **Fix:** rewrite the three agent files to describe the actual layout, and collapse the duplicated content — they are near-identical, so they should be one file plus two thin pointers.

### A2. The reference project does not follow the guideline

`sax-llm/pyproject.toml` is the concrete contradiction:

| Guideline says | `sax-llm` actually does |
| --- | --- |
| "mypy strict is a build gate… run it in CI" | **No mypy and no pyright at all.** |
| "Target a current Python (3.13+, ideally 3.14)" | `requires-python = ">=3.12"` |
| Lean on `hypothesis` for invariants/round-trips | **`hypothesis` is not a dependency.** |
| Mass of tests in cheap pure unit tests | `--cov-fail-under=25` (a 25% floor) |
| ruff enforces the mechanical rules | `select = ["E","W","F","I","N","UP","B","SIM","TCH","RUF"]` — missing the rules that enforce the prose (see A3/B5) |

Either the guideline is aspirational fiction or the project is out of compliance. For a "regulated trading shop" (the doc's own framing) the gap between documented control and actual control is itself the finding. **Fix:** bring `sax-llm` up to the gate (add the `[tool.mypy]` block from C5, the expanded ruff set from C4, `hypothesis`, and a realistic coverage floor), or down-scope the guideline's claims. The former.

### A3. The process docs verify syntax, not contracts

`make_todo.md`'s acceptance criteria are the weakest link in the whole chain, and they directly contradict `python-guidelines.md`:

- New-file acceptance is `python -m py_compile path/to/filename.py` — that proves the file *parses*, nothing more. A file full of type errors and broken contracts passes.
- Modification acceptance is `mypy path/to/existing.py` — single-file, **non-strict**, contradicting "mypy strict is a build gate."
- Coverage target is ">90%" (line coverage), which is both contradicted by `sax-llm`'s 25% and is the wrong metric (see B8).

Acceptance criteria are precisely where contracts should be *checked*. **Fix:** see C9 — upgrade the criteria to "ruff clean, `mypy --strict` clean on the package *and* tests, the stated property holds under `hypothesis`, and `crosshair check` finds no counterexample on the pure function."

### A4. The PEP 649 explanation conflates two different PEPs

`python-guidelines.md` line 63 says to drop `from __future__ import annotations` because "On Python 3.14+ (PEP 649) annotations are evaluated lazily by default… and because it *stringizes* annotations it breaks runtime-annotation consumers like pydantic and FastAPI."

The **advice is correct**, but the **reason is muddled** and will confuse a beginner. The *stringizing* behavior is PEP **563** (`from __future__ import annotations`), which replaces every annotation with a string and is what breaks pydantic/FastAPI. PEP **649** (3.14) is the *fix*: it makes annotations lazy but they evaluate to **real objects** on access, so it does **not** break pydantic. The sentence reads as though PEP 649 is the thing that stringizes. **Fix:** "PEP 563's `from __future__ import annotations` stringizes annotations, which breaks runtime consumers like pydantic/FastAPI. PEP 649 (3.14) makes annotations lazy *without* stringizing, so the future-import is now both redundant and harmful — drop it."

### A5. Internal duplication in a doc that preaches DRY

The `NewType` guidance appears twice, nearly verbatim (the "distinguishes IDs but does not validate" bullet under *Type system discipline* and "`NewType` for IDs everywhere" under *Make illegal states unrepresentable*). The "time/randomness/UUIDs… are pseudo-I/O" rule appears three times (module structure, side-effects, test strategy). Some repetition across sections is deliberate reinforcement, but a doc that mandates "don't repeat yourself" should consolidate these to one canonical statement plus a cross-reference. Minor, but it is the kind of thing this very repo flags in code review.

---

## Part B — Where the guidelines are weak (present but too thin for the goal)

### B1. "Contract" is used to mean "type signature," and never defined

The word "contract" appears a dozen times but always as a synonym for "the type signature." The new engineer never learns the actual vocabulary of contract-driven development:

- **Precondition** — what the *caller* must guarantee before calling (`qty > 0`).
- **Postcondition** — what the *function* guarantees on return (`result is sorted`, `0 <= pct <= 1`).
- **Invariant** — what is always true of an object between calls, or of a loop on every iteration (`self.low <= self.high`).

Types express *some* contracts (the shape of data). They cannot express "the result list is sorted," "splits sum to the input total," "this function performs no I/O," or "`price > 0`". Those are the contracts a contract-driven process exists to capture, and the doc is silent on all of them. **Fix:** add the section in C1/C2.

### B2. The `assert` statement — and its biggest footgun — is missing

`assert` is the cheapest contract tool in the language and the doc never mentions it. Worse, it never warns about the trap that bites every beginner: **`python -O` (and `PYTHONOPTIMIZE`) strips all `assert` statements and `if __debug__:` blocks.** Therefore:

- `assert` is correct for **internal invariants and postconditions in the pure core** — conditions you believe are *always* true and want checked in dev/CI.
- `assert` is **never** acceptable for validating external input, enforcing authorization, or any check that must survive in production — because in production it may not run at all. Those belong to parse-at-the-boundary (pydantic) and explicit `if … raise`.

A new engineer who validates a request body with `assert` ships a security hole the moment someone sets `PYTHONOPTIMIZE=1`. This must be stated explicitly.

### B3. The type-narrowing toolkit is incomplete

The doc mentions `TypeGuard`, `@override`, and `assert_never` — good — but for the stated goal ("maximize what static analysis can catch") it omits the sharper tools:

- **`TypeIs` (PEP 742, 3.13+) over `TypeGuard`.** The doc targets 3.13+, so this matters: `TypeGuard` narrows the type only in the `if` branch; `TypeIs` narrows in **both** branches (`if`/`else`), which is what people almost always want and what eliminates a class of "the checker still thinks it might be `None` in the else" bugs. New code should default to `TypeIs`; reserve `TypeGuard` for the rare asymmetric case.
- **`assert_type` and `reveal_type`.** `assert_type(x, Decimal)` is a *compile-time* assertion that the inferred type is what you think — it turns "I assume mypy sees this as `Decimal`" into a checked fact, and is invaluable in tests of generic/overloaded code. `reveal_type(x)` is the debugging counterpart.
- **`@final`** (on classes/methods that must not be subclassed/overridden) alongside the already-mentioned `@override`. Together they let the checker police your inheritance contracts.
- **`LiteralString` (PEP 675)** for injection-sensitive sinks (SQL string builders, shell commands) — it lets the checker reject a runtime-built string where only a literal is safe. Relevant for a trading shop touching databases.

### B4. The mypy configuration is under-specified — "strict" is not "strictest"

The doc lists `strict = true` plus a few flags, several of which (`warn_return_any`, `warn_unused_ignores`, `warn_redundant_casts`, `strict_equality`) are **already implied by `strict`** — so the list reads as more than it adds. Meanwhile it omits the high-value options that `strict` does *not* turn on:

- **`warn_unreachable = true`** — flags dead code, which is frequently a *symptom* of a wrong narrowing or a contradiction the type system found. This is one of the best "free bug finder" flags and it is not in `strict`.
- **`enable_error_code = [...]`** — the optional codes are off by default and are exactly the strictness a contract-driven shop wants: `redundant-expr`, `possibly-undefined`, `truthy-bool`, `truthy-iterable`, `ignore-without-code`, `unused-awaitable`, `mutable-override`, `explicit-override`, `redundant-self`, `deprecated`, `unimported-reveal`.
- **Type-check the tests too.** The worked example claims `mypy --strict` passes "on both files," implying tests are checked — but it is never stated as a rule, and tests are exactly where untyped helpers and stray `Any` breed.
- **A policy for untyped dependencies.** `anthropic`/`mistralai` may ship partial or no type information. The doc should say: prefer typed libraries; otherwise add `types-*` stubs, or wrap the dependency behind a typed adapter at the shell boundary (which the FC/IS pattern already pushes you toward). Without this, `Any` leaks in from the edges and silently defeats `--strict`.

### B5. The linter is told to "enforce the rules" but isn't given the rules

The doc's principle — "many rules below are mechanically enforceable… so let the linter enforce them instead of relying on review" — is exactly right, and then the ruff selection (`E,W,F,I,N,UP,B,SIM,TCH,RUF`) leaves several of its own prose rules unenforced. The prose says "never bare except," "timezone-aware datetime only," "no boolean soup," "async lives at the boundary," "keep functions short" — and none of those is wired. See C4 for the mapping and the expanded set.

### B6. "Contracts at the boundary" (pydantic) is thinner than it should be

The doc correctly says "parse loose external input into precise types once at the boundary," but the *boundary contract* itself is underspecified for a beginner. It should show:

- `model_config = ConfigDict(extra="forbid", frozen=True)` — `extra="forbid"` turns an unexpected/misspelled field into a validation error instead of a silently-ignored one (a real contract), and `frozen=True` keeps boundary models immutable like the core's dataclasses.
- `Annotated[int, Field(gt=0)]`, `Annotated[str, StringConstraints(...)]` — the runtime constraints *are* the precondition for the data entering the system. This is where "validate" lives, so that the core can "parse, don't validate" and trust the types thereafter (validate once, never re-check).

### B7. hypothesis is recommended but disconnected from contracts

Property-based testing is mentioned (good) but presented as a standalone testing trick. It is actually the *runtime sampler* for the contracts in B1: a postcondition like "splits sum to the total" or "`parse(render(x)) == x`" is a property, and `hypothesis` searches for a counterexample by sampling. The doc should draw the ladder (C1) so the new engineer sees types, asserts/contracts, hypothesis, and CrossHair as four rungs of the *same* idea, not four unrelated tools.

### B8. There is no way to know whether the tests/contracts are meaningful

Coverage (line coverage, the `make_todo.md` ">90%") tells you which lines *ran*, not whether your assertions would *catch a bug*. A test suite can have 100% coverage and assert nothing useful. The contract-driven answer is **mutation testing** (B8 → C7): deliberately break the code and confirm a test fails. The doc has no notion of this, so "our tests are good" is currently an article of faith.

---

## Part C — What to add (with paste-ready specifics)

### C1. A "four layers of contract" mental model (new section for `python-guidelines.md`)

Lead the contracts discussion with one picture so a beginner sees the whole strategy. Each layer catches what the layer above cannot, and gets more expensive as you go down — so push as much as possible upward:

1. **Types — static, free, always on.** Make illegal states unrepresentable (the doc already covers this well). Caught by mypy/pyright with zero runtime cost.
2. **Contracts / asserts — runtime, cheap.** The truths types can't express: `qty > 0`, "result is sorted", "splits sum to the total". A plain `assert` in the pure core, escalating to `icontract`/`deal` for public APIs (C2).
3. **Property tests — hypothesis.** Search the input space for a counterexample to those contracts. Cheap because the core is pure (the architecture already guarantees this).
4. **Symbolic verification — CrossHair.** Instead of *sampling* inputs, *prove* the contract holds (or hand you the exact input that breaks it) by exploring paths with an SMT solver (C3).

The key insight to give the beginner: **the Functional Core / Imperative Shell split is what makes layers 3 and 4 possible.** hypothesis and CrossHair both need deterministic, side-effect-free functions. The reason the doc bans I/O from the core pays off *here*.

### C2. Design by Contract: `assert`, then `icontract`/`deal`

Add explicit guidance:

- **Default to `assert` for internal invariants and postconditions in the pure core**, with the `python -O` warning from B2 stated loudly. Example on the existing shapes:

  ```python
  def vwap(first: Fill, *rest: Fill) -> Decimal:
      fills = (first, *rest)
      total_qty = sum(f.qty for f in fills)
      assert total_qty > 0, "non-empty by signature, positive by Fill invariant"
      price = sum(f.price * f.qty for f in fills) / total_qty
      assert price >= 0  # postcondition: a VWAP of non-negative prices is non-negative
      return price
  ```

- **Escalate to a contract library when** the contract is part of a *public* core API, needs to be inherited (Liskov-correct), or you want it machine-checked by CrossHair and turned into generated tests. Two options, both supported by CrossHair:
  - **`icontract`** — `@require` / `@ensure` / `@invariant`, informative violation messages, and (per its maintainers) the only Python DbC library that inherits contracts correctly (weakening preconditions / strengthening postconditions per LSP). Best when you have class hierarchies.
  - **`deal`** — `@deal.pre` / `@deal.post` / `@deal.ensure` / `@deal.inv`, **plus `@deal.raises(...)` and `@deal.has(...)`** which declare *which exceptions a function may raise* and *whether it performs I/O*. That directly fixes the doc's own lament that "a signature says nothing about what it raises (Python has no checked exceptions)." `deal` also ships a linter and generates hypothesis-backed tests from the contracts. Best when you want the exception/side-effect contract made explicit and statically checked — a strong fit for the FC/IS boundary.

  ```python
  import deal

  @deal.pre(lambda fills: len(fills) > 0)
  @deal.ensure(lambda fills, result: result >= 0)
  @deal.has()                 # declares: pure, no side effects — statically checked
  @deal.raises()              # declares: raises nothing — statically checked
  def vwap(fills: Sequence[Fill]) -> Decimal: ...
  ```

- **Decision rule for the beginner:** plain `assert` in the core by default → `deal` when you want the raises/side-effect contract enforced or auto-generated tests → `icontract` when contract *inheritance* matters. Don't mix two contract libraries in one module.

### C3. CrossHair: the static-analysis tool that verifies contracts

This is the single highest-leverage addition for "maximize what static analysis can catch," and it is one of the three tools the task already has available.

- **What it is:** an analysis tool that executes your functions on *symbolic* inputs and uses an SMT solver to search execution paths for one that violates a contract — a type, a plain `assert`, or an `icontract`/`deal` condition (all checked by default). When it finds one, it prints the **concrete counterexample** input, which is far more actionable than a coverage number.
- **Why it pairs with this architecture:** CrossHair is most effective on **deterministic, side-effect-free** code, because it intercepts and reasons about pure computation but cannot meaningfully explore real I/O, randomness, or the network. That is *exactly* the functional core. The doc's whole FC/IS discipline is what makes CrossHair usable — say so.
- **How to run it (low friction):** `crosshair check path/to/core_module.py` in CI on the pure modules only; `crosshair watch` in the editor for live feedback while writing a pure function; `crosshair diffbehavior old new` to prove two implementations are equivalent — a refactor safety net that complements "a refactor shouldn't require editing tests."
- **Honest limits to set expectations:** it is slower than mypy (it's a solver), it is bounded (it won't explore unbounded loops fully), and it only shines on the pure core — so it is a *targeted* gate on `core/`, not a whole-repo blanket. Run it on the modules where correctness is monetary.

### C4. The ruff rule set that actually enforces the prose

Replace the `sax-llm` selection with one that mechanizes the doc's own rules. Each addition below corresponds to a sentence already in `python-guidelines.md`:

```toml
[tool.ruff.lint]
select = [
  "E", "W", "F", "I", "N", "UP", "B", "SIM", "RUF",  # current set
  "TC",     # type-checking imports under TYPE_CHECKING  -> "type-only imports go under if TYPE_CHECKING"
  "ANN",    # missing annotations                        -> "include type hints for parameters and return values"
  "BLE",    # blind-except (BLE001)                       -> "Catch narrow, and never bare"
  "TRY",    # tryceratops, exception anti-patterns        -> error-handling section (raise ... from, no broad catch)
  "EM",     # exception message hygiene                   -> "message carries the human detail"
  "DTZ",    # naive datetime                              -> "timezone-aware datetime only — never naive"
  "FBT",    # boolean-trap                                -> "replace boolean/optional soup with a state union"
  "ASYNC",  # flake8-async                                -> "async lives in the shell / don't propagate async"
  "C90",    # mccabe complexity (set max-complexity)      -> keeps functions within reach of review + CrossHair
  "PL",     # pylint (esp. PLR refactor, PLE errors)      -> general correctness/refactor
  "PT",     # pytest style                                -> test-strategy section
  "S",      # bandit security                             -> "regulated trading shop"
  "G", "LOG",  # logging format/usage                     -> "logging is a side effect … at the shell"
  "RET", "ARG", "PIE", "TID", "PERF", "FURB",  # general cleanliness/perf
]

[tool.ruff.lint.mccabe]
max-complexity = 10           # see C6

[tool.ruff.lint.flake8-type-checking]
runtime-evaluated-base-classes = ["pydantic.BaseModel"]  # don't TYPE_CHECKING-guard pydantic models
```

Note the last block: `TC` (flake8-type-checking) would otherwise try to move pydantic model imports under `if TYPE_CHECKING:`, which breaks runtime validation — the config above prevents the linter from fighting the "drop `from __future__ import annotations`" rule. This interaction is itself worth a sentence in the doc.

### C5. The complete mypy gate (and a pyright note)

```toml
[tool.mypy]
python_version = "3.13"
strict = true
warn_unreachable = true        # NOT in --strict; catches dead code from bad narrowing
enable_error_code = [
  "redundant-expr", "possibly-undefined", "truthy-bool", "truthy-iterable",
  "ignore-without-code", "unused-awaitable", "mutable-override",
  "explicit-override", "redundant-self", "deprecated",
]
files = ["src", "tests"]       # type-check tests too
# packages without type info: add stubs or a typed adapter; do not blanket-ignore.

[[tool.mypy.overrides]]
module = ["mistralai.*"]       # example: an untyped dependency, scoped narrowly
ignore_missing_imports = true
```

`pyright --strict` is an equally valid authoritative gate; pick one and run it in CI. The fast checkers `ty` (Astral) and `pyrefly` (Meta) are good for the in-editor inner loop, but keep mypy/pyright as the gate until they reach parity — the doc already says this; just make sure CI runs the gate on `src` **and** `tests`.

### C6. Complexity and duplication gates (answers `radon` and `jscpd`)

**Why complexity matters for a *contract* shop, specifically:** every branch multiplies the paths a human reviewer, a type narrower, and CrossHair's solver must consider. A function with cyclomatic complexity 30 is effectively unverifiable — CrossHair times out, narrowing gets fragile, and bugs hide in the path explosion. Capping complexity keeps functions *within reach of the tools you're trying to maximize.*

- **`radon`** for measurement: `radon cc -s -n C src` (cyclomatic complexity, flag grade C and worse), `radon mi -n B src` (maintainability index). Good for reports and trend-watching.
- **`xenon`** to make it a CI gate (radon doesn't fail a build by itself): `xenon --max-absolute B --max-modules A --max-average A src`. ruff's `C901` covers the same ground inline for the everyday loop; use both — ruff in pre-commit, xenon in CI.
- **`jscpd`** for copy-paste detection: duplicated logic is duplicated *bug surface*, and the classic failure is fixing a contract in one copy and missing the other two. `jscpd` is language-agnostic, which suits this polyglot repo (Python + TypeScript + nushell): `jscpd --min-tokens 50 --threshold 1 --reporters console,html src`. Run it as a non-blocking report first, then ratchet the threshold down.

### C7. Mutation testing (answers "are my contracts real?")

Coverage says a line *ran*; mutation testing says a test would *catch a bug there*. Add it as the trust check on the pure core:

- **`mutmut`** for most projects (simple, line-based, `# pragma: no mutate` to exclude). **`cosmic-ray`** when you need AST-level mutants, custom operators, or parallel/distributed runs on a large core.
- It is **slow** — run it on `core/` on a schedule (nightly / pre-release), not on every commit. The everyday floor stays cheap: enable **branch coverage** (`--cov-branch`) and raise `sax-llm`'s `--cov-fail-under` from 25 toward the regime the doc implies (the core should be near-exhaustively covered because it's cheap; the shell carries the integration tests).

### C8. The friction story: one gate, one command, run locally first

The task wants to *minimize friction*. The answer is to make the whole gate runnable in one step and to fail in the editor, not in CI:

- A **`pre-commit`** config running `ruff` + `ruff format --check` + `mypy` (and `crosshair check` on changed core modules) so problems surface before a push.
- A single task — `uv run` script or a `Makefile`/`justfile` target — that runs ruff, mypy, pytest, xenon, and (on demand) crosshair, identical locally and in CI. "Reproducible across machines and CI" is already a stated value; this is how you cash it.
- Fast in-editor checking (`ty`/`pyrefly`, `crosshair watch`) for the inner loop; the slow authoritative gates (mypy/pyright, xenon, mutation) in CI.

### C9. Make contracts start at the spec (process docs)

`make_spec.md` and `make_todo.md` predate the type discipline — they speak in "validation rules" and JSON examples and never mention encoding invariants as types. For contract-driven development, the **spec is where contracts are born**:

- `make_spec.md`: add a step that, for each operation, captures **preconditions, postconditions, and invariants** explicitly (not just a flat "validation rules" list), and a data-model step that asks "what illegal states does this model permit, and how do we make them unrepresentable?" The spec should name the *properties* (round-trips, conservation laws) that will become hypothesis/CrossHair checks.
- `make_todo.md`: replace the `py_compile` / single-file-`mypy` acceptance criteria (A3) with **contract-grade** criteria:

  ```text
  - Acceptance:
    - Verification: `ruff check`, `mypy --strict` (src + tests), and `uv run pytest` all pass
    - Property: <stated invariant> holds under `hypothesis`
    - Verification: `crosshair check <module>` finds no counterexample   # pure-core tasks
    - Metrics: no new mypy errors; complexity grade <= B (xenon)
    - Failure: type errors, contract counterexample, unhandled raise not in the declared set
  ```

  And reconcile the coverage target with reality (C7) so the spec and the project agree.

---

## Part D — Tool recommendation summary

Answering the explicit question. The three offered tools are all worth using; the gaps are filled by a small set of additions.

| Tool | Verdict | Role |
| --- | --- | --- |
| **CrossHair** | **Adopt — headline** | Symbolic verification of types/asserts/contracts on the **pure core**; prints concrete counterexamples. The payoff of the FC/IS discipline. (C3) |
| **radon** (+ **xenon**) | **Adopt** | Complexity/maintainability measurement; `xenon` makes it a CI gate. Keeps functions within reach of review and CrossHair. (C6) |
| **jscpd** | **Adopt (modest)** | Cross-language copy-paste detection — fits this polyglot repo; duplicated logic = duplicated bug surface. (C6) |
| **mypy** *or* **pyright** | **Adopt — currently missing** | The authoritative type gate the doc *claims* exists but `sax-llm` lacks. (C5) |
| **ruff (expanded set)** | **Adopt** | Mechanize the prose rules the current selection leaves unenforced. (C4) |
| **icontract** *or* **deal** | **Adopt** | Runtime Design-by-Contract; both checked by CrossHair. `deal` also gives raises/side-effect contracts + generated tests. (C2) |
| **hypothesis** | **Adopt — currently missing as a dep** | Property-based sampling of contracts; the doc recommends it but the project doesn't install it. (B7) |
| **import-linter** | **Adopt — high value, non-obvious** | Mechanically enforces "imports go inward, never reverse" as a CI **contract** (Layers/Forbidden contracts). The doc's #1 architectural rule is currently honor-system only; this makes the FC/IS boundary unbreakable. |
| **mutmut** / **cosmic-ray** | **Adopt (scheduled)** | Mutation testing — proves the tests/contracts actually catch bugs. Run on the core, nightly. (C7) |
| **pre-commit** | **Adopt** | Orchestrate ruff/mypy/crosshair locally; the friction-reduction backbone. (C8) |
| **pip-audit** | **Consider** | Dependency vulnerability scan — appropriate for a regulated shop; complements the `uv.lock` reproducibility story. |
| **interrogate** | **Optional** | Docstring-coverage gate, if public-API documentation becomes a requirement. |

The standout non-obvious recommendation is **import-linter**: the guidelines rest entirely on the Functional Core / Imperative Shell boundary, yet that boundary is enforced only by reviewer vigilance today. `import-linter` turns "pure modules must not import I/O modules" into a contract the build checks on every commit — arguably the highest-leverage single addition after wiring mypy.

---

## Suggested sequencing (don't boil the ocean)

For a new engineer or a team adopting this incrementally:

1. **Close the credibility gap first (low effort, high signal):** add the `[tool.mypy]` gate (C5) and expanded ruff set (C4) to `sax-llm`, wire `pre-commit` (C8), fix the agent files (A1). Now the reference project matches the guideline.
2. **Make the architecture self-enforcing:** add the `import-linter` contract for the FC/IS boundary.
3. **Introduce contracts on the core:** start with plain `assert` postconditions (C2) and run `crosshair check` on one pure module (C3) to feel the payoff.
4. **Add the quality gates:** `xenon` complexity (C6), branch coverage + a realistic floor (C7), `jscpd` as a report.
5. **Deepen where it pays:** `deal`/`icontract` on rich public APIs, `hypothesis` properties for monetary/parsing logic, mutation testing on a schedule.
6. **Fold the conceptual material back into `python-guidelines.md`** (C1) and upgrade the process docs' acceptance criteria (C9) so the next project starts contract-first.

---

## Sources

- [CrossHair — kinds of contracts (asserts, PEP 316, icontract, deal)](https://crosshair.readthedocs.io/en/latest/kinds_of_contracts.html) and [project README](https://github.com/pschanely/CrossHair)
- [icontract — design-by-contract with inheritance](https://github.com/Parquery/icontract) and [docs](https://icontract.readthedocs.io/en/latest/introduction.html)
- [deal — DbC, linter, exception/side-effect contracts, test generation](https://github.com/life4/deal) and [formal verification](https://deal.readthedocs.io/basic/verification.html)
- [PEP 316 — Programming by Contract for Python](https://peps.python.org/pep-0316/)
- [PEP 742 — Narrowing types with `TypeIs`](https://peps.python.org/pep-0742/) and [TypeIs vs TypeGuard, explained](https://rednafi.com/python/typeguard-vs-typeis/)
- [mypy — error codes for optional checks (`enable_error_code`)](https://mypy.readthedocs.io/en/stable/error_code_list2.html) and [common issues (`warn_unreachable`, `strict_equality`)](https://mypy.readthedocs.io/en/stable/common_issues.html)
- [Ruff — rules index](https://docs.astral.sh/ruff/rules/) — `BLE` (blind-except), `TRY` (tryceratops), `DTZ`, `FBT`, `ASYNC`, `C901`, `ANN`, `S`, `PL`
- [Cosmic Ray — mutation testing for Python](https://cosmic-ray.readthedocs.io/) and [mutmut](https://medium.com/hackernoon/mutmut-a-python-mutation-testing-system-9b9639356c78)
- [Alexis King — Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) (already cited by `python-guidelines.md`; the conceptual basis for "validate once at the boundary")
