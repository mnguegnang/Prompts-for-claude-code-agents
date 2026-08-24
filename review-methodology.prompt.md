# AI Review Methodology, Evaluation & Observability Standards

Cross-cutting review standards applied to ALL AI project reviews regardless of domain. Loaded
alongside any domain-specific framework.

**Always load alongside this file:** `~/.claude/prompts/evaluation-methodology.prompt.md`. That
file defines what a trustworthy number looks like; this file defines how you review one.

## 0. Evidence Standard (applies to every finding)

- A **`Confirmed`** finding of severity **Critical or High** must ship with a **reproduction**: a
  failing test, a log line, a command with its output, or an executed snippet. Without one it is
  downgraded to `Likely`, regardless of how obvious it looks.
- `Likely` — strong inference from code patterns, incomplete context.
- `Uncertain` — requires user clarification. Never fix without confirmation.
- **Never accept a self-reported gate result.** A `HANDOFF` block's "Quality gates: <verbatim
  results>" is an unverified string. Re-run the gate sequence yourself in Phase 1 — it is cheap,
  and it converts a claim into an observation.

## 1. Experiment Comparison Intelligence

If the user asks "which approach is better?", verify the comparison is scientifically valid. Both
approaches must share: same data splits, batch size, hardware, seeds, stopping criteria,
preprocessing pipeline, and evaluation cadence.

**Statistical reporting.** Averaging across runs does **not** establish significance. Require:
- **n stated explicitly.** n=1 is an anecdote and must be labelled as such.
- **Dispersion reported** — stratified bootstrap confidence intervals and interquartile mean (IQM)
  rather than bare mean/median point estimates. `rliable` implements this for the small-n regime
  that ML work actually operates in.
- **Effect size**, not just the direction of the difference.

Flag as **High severity**: any comparison lacking shared conditions, any result reported without
dispersion, and any claim of "significance" supported only by a mean.

## 2. Evaluation Setup

AI projects fail at evaluation, not at implementation. Check:
- Metrics match the product goal (ROC-AUC/F1 for classification, NDCG/MRR for retrieval, tool
  success rate and pass^k for agents).
- **Pre-registered acceptance criteria exist** — baseline, primary metric, minimum improvement,
  guardrails, kill criterion — and were fixed before implementation. Thresholds moved after seeing
  results, without a logged deviation, are **Critical** (the definition of "done" was fitted to the
  outcome).
- Offline evaluation aligns with real-world usage; each metric maps to a business outcome.
- Baselines are meaningful and benchmark comparisons fair.
- Test sets are truly held-out — never trained on, tuned on, or used for prompt iteration — and
  checked for pretraining contamination where a foundation model is involved.
- Results are reported **per slice**, not only in aggregate. An aggregate win that reverses on a
  slice is not a win.

## 3. Error Analysis — inspect outputs, not just metrics (mandatory for GenAI/agentic reviews)

Inspecting `df.describe()` tells you the shape of the summary and nothing about how the system
fails. Before or alongside metric review, run the §3 protocol from
`evaluation-methodology.prompt.md`:

1. Sample ~30 real traces with full context (input, retrieved chunks, tool calls, output).
2. Binary pass/fail per trace plus a written critique.
3. Open-code failures into a taxonomy; continue until no new modes appear.
4. Produce a frequency table and **prioritise findings by it.**

A review of a GenAI or agentic system that never read the system's own outputs is incomplete —
say so explicitly in the report rather than implying full coverage.

## 4. Eval Artifact Inspection

When evaluation outputs exist, **inspect them directly** — do not rely solely on the code:
- **CSV / JSON lines:** load with pandas or jq and verify reported metrics match the actual data.
- **W&B / MLflow:** check run configs, hyperparameter logging completeness, metric consistency.
- **TensorBoard:** verify loss curves are not diverging and LR schedules match code.
- **Training logs:** inspect for NaN losses, gradient explosion, OOM, or suspiciously flat metrics.
- Flag any mismatch between code logic and reported results as **Critical** (possible leakage or
  evaluation bug).
- **Judge outputs:** if an LLM judge produced any reported metric, verify it was validated per §6
  of `evaluation-methodology.prompt.md` (precision/recall against held-out human labels,
  temperature 0, position randomised). An unvalidated judge is **High** — the reported number is
  not evidence.

## 5. Workflow & Observability

Code is only half the system. Require instrumentation across these dimensions:

- **Performance:** TTFT, TPOT, throughput (TPS), cost per inference.
- **Quality & safety:** format-failure rate, factual consistency, toxicity, **false refusals**.
- **User engagement:** early termination, conversation turn count, regeneration requests — the
  cheapest real-world quality signal and the one most often absent. Flag its absence.
- **Component health:** retriever hit rate and latency, tool-selection distribution, cache hits.
- **Traceability:** per-step tracing in agent systems; tags allowing step-by-step replay of any
  single query.
- **Governance:** experiment tracking, data versioning, checkpoint strategy, rollback readiness.
- **Rollout:** shadow/A-B mechanism, live guardrail metrics, and an explicit rollback trigger.
  Serving code with no rollback condition is **High**.

## 6. Software Security Checklist

ML projects are software projects. Apply to every review.

**Run the tools; do not eyeball these.**
- `pip-audit -r requirements.txt` — dependency CVEs. `bandit` is an AST scan of your own code and
  never reads `requirements.txt`; the dependency class is unguarded without this.
- `gitleaks detect --no-git --no-banner --redact` — secrets in the working tree. Any hit is
  **Critical**. `--no-git` is required: without it gitleaks scans commit history only and misses
  uncommitted secrets, and it errors outright outside a git repository. Sweep history separately
  with `gitleaks detect --no-banner --redact` when reviewing an inherited repo.

### Secrets & Credentials
- Verify `.env` is in `.gitignore`. A committed `.env` is **Critical**.
- Secrets loaded from environment or a secrets manager, never from literals.

### Dependency Security
- Pinned versions in `requirements.txt` / `pyproject.toml`. Unpinned is **Medium**.
- `pip install --trusted-host` or `--no-verify` (disabled TLS) is **Critical**.

### Input Validation & Serialization
- `pickle.load()`, `torch.load()` without `weights_only=True`, or `yaml.load()` without
  `SafeLoader` — **Critical** (arbitrary code execution).
- Path traversal (`../`) in user-supplied paths.
- Input validation and sanitisation at every system boundary.

### Network & File Access
- `subprocess.run(shell=True)` or `os.system()` with user-controlled input — **Critical**.
- `eval()` / `exec()` on untrusted data — **Critical**.
- HTTP requests use timeouts and validate TLS certificates.

### Data & Model Licensing
- Every dataset and model has a recorded licence. Verify the licence permits the stated
  deployment (non-commercial assets in a production build are **High**; unknown provenance is
  **High**).
- Check for PII in data entering model context and for masking where the blueprint requires it.

## 7. Severity Definitions

| Severity | Definition |
|----------|-----------|
| **Critical** | Security flaw, data leakage, invalid evaluation, results-fitted acceptance criteria, dangerous agent behavior |
| **High** | Major performance issue, unvalidated judge, missing dispersion, serious maintainability risk, reproducibility failure, licence violation |
| **Medium** | Weak abstractions, test gaps, surviving mutants, inefficiency, code smell |
| **Low** | Style issues, minor duplication, naming improvements |

**Trade-off empathy still applies:** weigh severity against stated project context (PoC vs.
production) before assigning Critical or High. Ask when intent is ambiguous.
