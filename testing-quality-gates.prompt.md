# Testing & Quality Gates (Coder Steps 4–5)

Testing protocol and quality gate enforcement for the Coder Agent.

> **Read first:** the five formatting/lint/type/test/security gates below check that code is
> well-formed. A change can pass all of them and still do the wrong thing, and `pytest` is
> grading tests written by the same agent that wrote the code. Gates 6–8 exist to break that
> loop. For whether the *system* is any good, load
> `~/.claude/prompts/evaluation-methodology.prompt.md` — that is a separate question from this file.

## Cost Protection

### Dry Run

Before any script that processes bulk data or makes looping calls to paid APIs:

1. Process exactly **ONE** item, display the result.
2. **Cost Estimation:** measure tokens (or API units) for the single item, multiply
   `unit_cost × total_items = estimated_total_cost`, and present:
   *"Dry run successful. Estimated cost for full run: $X.XX (Y items × $Z.ZZ each). Approve full execution?"*

### Enforced Budget Guard (not an estimate — a limit in the code)

An approved estimate is not a control. Retries, longer-than-sampled contexts, and agent loops
routinely exceed it (OWASP LLM10 — Unbounded Consumption). Every generated pipeline that calls a
paid API or runs an agent loop **must ship with a runtime guard**:

- A token/spend counter incremented on every call, reading its ceiling from `config.yaml`.
- A hard **raise** (not a warning) when the ceiling is hit, with the partial result persisted.
- **Every agentic loop carries both `max_iterations` and `max_spend`.** A loop with only an
  iteration cap is unbounded in cost.
- Ceilings are config, never hardcoded. Log cumulative spend per run.

Code that calls a paid API without a budget guard does not pass Step 5.

## Test Layers

### Layer 1 — Conventional (always run, always mocked where external)

- **Unit:** utility functions and data-processing modules.
- **Integration:** pipeline component connections.
- **Smoke:** end-to-end minimal execution.
- **API:** FastAPI endpoint validation (if applicable).
- **GenAI structural:** prompt/template rendering, retrieval output shapes, tool routing logic,
  Pydantic schema conformance.

### Layer 2 — Property-based & metamorphic (required for any ML/GenAI logic)

Conventional tests assert known outputs. **ML code frequently has no known-correct output** — the
oracle problem. Asserting relations between outputs works where asserting outputs cannot.

- **Property-based (Hypothesis):** tensor shape/dtype invariants, data-pipeline round-trips,
  chunker boundary behaviour, config parsing. This is where shape and dtype bugs actually live.
- **Metamorphic relations** — instantiate at least three relevant to the domain:
  - Permuting training-set order must not change final metrics beyond seed noise.
  - Duplicating a retrieved document must not change the grounded answer.
  - Scaling all logits by a positive constant must not change `argmax`.
  - Adding an irrelevant document to RAG context must not change the answer.
  - Reordering retrieved chunks must not change a faithful answer's claims.
  - Semantically-equivalent paraphrase of the input must not flip a classification.

### Layer 3 — Provider contract tests (mocks alone are insufficient)

Mocking every LLM call means the provider contract is **never** exercised: prompt-format errors,
token-limit boundaries, tool-schema mismatches, and streaming edge cases all pass silently
against a fake the same agent wrote. This layer therefore has 3 **modes** (not to be confused with
the 3 test layers above):

1. **Mocked** — the bulk suite. Fast, free, always run.
2. **Recorded cassettes** — real responses recorded once, replayed free in CI, via
   `pytest-recording` (which wraps `vcrpy`) or `vcrpy` directly. Re-record on any provider or SDK
   version bump. This is the contract test. Scrub API keys from cassettes with VCR's
   `filter_headers`/`filter_post_data_parameters` before they hit the repo.
3. **Live smoke** — at least one real call per provider, behind an explicit flag
   (`pytest -m live`), run before Step 6 and after any dependency bump.

The cost objection justifies the mocked mode as the default. It does not justify zero real calls ever.

Run everything except live calls: `pytest tests/ -v -m "not live"`

**Register the marker or this silently degrades.** An unregistered `live` marker raises
`PytestUnknownMarkWarning`, and under `--strict-markers` it is a hard error. Add to `setup.cfg`:

```ini
[tool:pytest]
markers =
    live: makes real API calls; excluded by default, costs money
```

**All 3 layers are required.** Layer 2 is the one most often skipped and the one that matters most
for ML code, because ML frequently has no known-correct output to assert against.

## Quality Gate Sequence

You are **strictly forbidden** from proposing a commit for code that has not passed all gates.

```bash
# 1–5: is the code well-formed?
black src/ tests/                       # author mode: rewrites files
ruff check src/ tests/
mypy src/
bandit -r src/ -q                       # AST scan of YOUR code
pytest tests/ -v -m "not live"

# 6: are the DEPENDENCIES safe?  bandit cannot see requirements.txt.
pip-audit -r requirements.txt           # OSV / PyPA advisory DB

# 7: are there secrets in the tree?  Do not eyeball this.
gitleaks detect --no-git --no-banner --redact   # or: trufflehog filesystem .

# 8: are the TESTS any good?  This grades the suite, not the code.
~/.claude/scripts/mutation-diff.sh          # diff-scoped; see below
mutmut results
```

If any gate fails: fix, log in `Developer_Log.md`, re-run.

### Author mode vs auditor mode

You run `black src/ tests/`, which **rewrites** files. Any independent re-verification runs
`black --check --diff src/ tests/`, which writes nothing and exits 0 (no change), 1 (would
reformat), or 123 (internal error). Both are correct for their purpose: you are the author and
should fix formatting, an auditor must observe without mutating what it audits.

### Which gates get independently re-verified

The `Stop` hook re-runs **7 of the 8**: black (in `--check` mode), ruff, mypy, bandit, pytest,
pip-audit, and gitleaks. Gate 8 is excluded deliberately, for 2 reasons:

- **Cost.** Mutation analysis runs the test suite once per mutant, so cost is roughly
  (suite runtime × mutant count). Google does not run full mutation analysis in the normal path
  for this reason, using a diff-scoped, mutant-sampled subset at code-review time instead
  (Petrović & Ivanković, ICSE-SEIP 2018). A blanket re-run at every session end contradicts that.
- **It is not a binary gate.** `mutmut` reports surviving mutants for a human to examine, not a
  pass/fail exit contract. Equivalent mutants exist and cannot be killed, so blocking on a nonzero
  exit would block on noise.

So gate 8 is **yours to run**, diff-scoped, with judgment applied to the survivors. Report the
mutation score or the falsification evidence explicitly; no hook will do it for you.

### Gate 6 — dependency vulnerabilities

`bandit` builds an AST from your source and runs plugins against it. It never reads
`requirements.txt`, so the entire dependency-vulnerability class is unguarded without `pip-audit`,
which checks each pinned version against the OSV and PyPA advisory databases. Never assess
dependency CVEs from memory — that is a lookup, not a judgement.

### Gate 7 — secrets

Scanning for hardcoded keys by reading the code is a task no agent should perform by eye.
`gitleaks`/`trufflehog` do it deterministically. A hit is **Critical**, always.

**`--no-git` is mandatory, not optional tuning.** Plain `gitleaks detect` scans *commit history*
and ignores the working tree, so it returns a clean exit on an uncommitted secret — which is
exactly the case here, since you write files and are forbidden from committing them. It also
exits non-zero in a directory that is not a git repository at all, blocking on projects that have
no history to scan. `--no-git` scans the filesystem, which is what this gate is for.

To additionally sweep history (worth doing once on an inherited repo, not per change):
`gitleaks detect --no-banner --redact`.

### Gate 8 — test effectiveness (the gate that breaks the self-grading loop)

Coverage is only weakly correlated with suite effectiveness once suite size is controlled
(Inozemtseva & Holmes); **assertion density** is strongly correlated (Zhang & Mesbah); and mutation
testing is the strongest available criterion, cheap enough to run per-diff (Petrović & Ivanković).

- Scope mutation to the **diff**, not the whole tree. Use `~/.claude/scripts/mutation-diff.sh`,
  which computes the scope for you. Full-tree mutation costs (suite runtime × mutant count) and is
  not affordable per change.

  **`--paths-to-mutate` does not exist in mutmut 3.x.** It was removed. mutmut 3 reads
  `source_paths` from `setup.cfg` and filters by positional MUTANT_NAMES, which are the module's
  *importable dotted path*, not a file path:

  ```
  src/mypkg/alpha.py     ->  mutant named  mypkg.alpha.x_add__mutmut_1
  src/mypkg/sub/deep.py  ->  mutant named  mypkg.sub.deep.x_deep__mutmut_1
  ```

  So diff-scoping means converting changed files to dotted module globs, which the script does:

  ```bash
  ~/.claude/scripts/mutation-diff.sh --print   # show the scope without running
  ~/.claude/scripts/mutation-diff.sh           # run it
  # emits, e.g.:  mutmut run "mypkg.alpha.*" "mypkg.beta.*"
  ```

  It excludes deleted files, covers committed, staged, and unstaged changes, exits cleanly when
  nothing relevant changed, and retries with a wildcard scope if the dotted name does not match
  your layout. **This requires `source_paths` in `setup.cfg`:**

  ```ini
  [mutmut]
  source_paths=src/<your_package>
  ```
- **Surviving mutants are findings.** Each one is a behaviour no test observes.
- If mutation testing is unavailable or too slow, the **falsification fallback is mandatory**:
  for every new test, break the corresponding source line, show the test fails, restore it.
  Record the observed failure in `Developer_Log.md`.

> **A test never observed failing has proven nothing.** Report the falsification evidence, not
> just "tests pass".

Explicitly reject the assertion antipattern: `assert result is not None`, `assert len(x) > 0`,
and bare `assert response` are not assertions about behaviour. Every test asserts a specific
expected value, relation, or raised exception.

### Zero-test detection

`pytest` exits 0 on a suite where every test is skipped or trivially passes. Before reporting a
pass, confirm the collected count is non-zero and matches the modules changed:
`pytest tests/ -q --collect-only | tail -1`. Report the number of tests collected alongside the result.

## Developer Log Summary

Before presenting the commit, output the tally from `Developer_Log.md`:
*"Developer Log: X errors encountered and resolved. Y open items remaining."*

## Git Review Loop

**You never commit.** Committing, pushing, tagging, and `git init` are the user's decisions and are
blocked at the tool level — a `PreToolUse` guard returns exit code 2 if you attempt one. Do not
retry a blocked command and do not route around the guard with another tool.

Once all gates pass:
1. Run `git status` and `git diff --stat`. Staging with `git add` is allowed.
2. Emit a `COMMIT MESSAGE` block for the user to run:

```
COMMIT MESSAGE
<subject line, imperative mood, ≤72 chars>

<body: what changed and why; reference Developer_Log entries and any approved deviation>

Command:
  git commit -m "<subject>" -m "<body>"
```

3. Hand it to the user and stop. Report **all eight** gate results, the tests-collected count, the
   mutation score or falsification evidence, and the Developer Log tally alongside it.
