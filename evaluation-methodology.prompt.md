# Evaluation Methodology (shared — PI Phase 4, Coder Steps 2.5/4, Reviewer Phase 2)

The cross-cutting evaluation standard for every AI system this pipeline designs, builds, or
reviews. Quality gates (`black`, `ruff`, `mypy`, `bandit`, `pytest`) check that code is
well-formed. **Nothing in that list checks that the system is any good.** This file does.

Load this file whenever you are: specifying an evaluation protocol (PI STAGE 2.5), building or
running an evaluation (Coder), or judging whether reported results mean anything (Reviewer).

---

## 0. The governing rule

> A metric produced by an uncalibrated instrument is not evidence.

Every number this pipeline reports must be traceable to (a) a stated eval set, (b) a stated
measurement method, and (c) a stated uncertainty. A number without all three is a `Likely`
finding at best, never `Confirmed`.

---

## 1. Evaluation-Driven Development (EDD)

Define the evaluation **before** building the thing being evaluated. The blueprint's evaluation
spec is written at STAGE 2.5, before the Baseline PoC — the PoC's purpose is to produce the
first number on the eval set, not merely to prove connectivity.

Specify capability across four buckets (Huyen, *AI Engineering*, Ch. 3–4):

| Bucket | Question | Typical measurement |
|---|---|---|
| Domain-specific capability | Does it know the domain? | Closed-ended benchmark, expert-labelled set |
| Generation capability | Is the output well-formed and faithful? | Faithfulness, format-validity rate |
| Instruction-following | Does it respect constraints? | Constraint-satisfaction rate, schema conformance |
| Cost & latency | Is it affordable and fast enough? | TTFT, TPOT, TPS, cost/inference |

**Do not chase 100%.** Saturating your own eval set is overfitting to your own ruler. State a
target, not a maximum.

## 2. Pre-registered acceptance criteria (mandatory)

Before implementation begins, the blueprint must fix, in writing:

- **Baseline to beat** — the naive/incumbent system and its measured score.
- **Primary metric** — one metric, named, with the eval set it is computed on.
- **Minimum improvement that counts** — the effect size below which the change is not worth shipping.
- **Guardrail metrics** — what must *not* regress (cost, latency, refusal rate, safety).
- **Kill criterion** — the result at which this approach is abandoned rather than tuned.

Thresholds are frozen once implementation starts; moving them after seeing results is
prohibited and must be logged as a deviation in `Project_Notes.md` with justification.

**Criteria drift caveat (Shankar et al., UIST 2024):** the *thresholds* are frozen, but the
*criteria taxonomy* is not. You cannot fully enumerate what "good" means before grading real
outputs. Revise the taxonomy from error analysis (§3), re-baseline, and record both versions.

## 3. Error analysis — look at the data (mandatory, do this first)

The highest-value activity available and the one most often skipped. Aggregate statistics
(`df.describe()`, a loss curve, a mean score) describe the summary; they never tell you *how*
the system fails. Open the outputs.

**Protocol:**

1. Sample **~30 real traces** from the target distribution (production if available, else the
   eval set). Include full context: input, retrieved chunks, tool calls, intermediate steps, output.
2. For each, make a **binary judgment** — did it achieve the desired outcome, yes or no — plus a
   **written critique** explaining the verdict in enough detail that a new team member could follow it.
3. **Open-code** the failures into a taxonomy of failure modes. Do not start from a predefined list.
4. **Continue sampling until new failure modes stop appearing.** That, not a fixed count, is the
   stopping rule. Complexity dictates volume.
5. Produce a **frequency table**: failure mode → count → % of sampled traces.
6. **Work is prioritised by that table**, not by intuition and not by what is easy to fix.

Specialised evals and judges are built **after** this step, targeting the modes it found — never before.

## 4. Binary judgments, not Likert scales

Use **pass/fail plus a written critique**. Do not use 1–5 quality scales.

A 1–5 scale has no anchored meaning — nothing defines what separates a 3 from a 4 — and it
produces metric sprawl no one can act on. A binary verdict forces an explicit decision about what
actually matters, and the critiques become the few-shot examples for any judge you later build
(Husain, *critique shadowing*).

Where stakeholders demand multi-dimensional scores, decompose into **several independent binary
questions**, each with its own pass rate. Never sum ordinal scales across dimensions and never
threshold the sum.

## 5. Choosing a measurement method (in preference order)

1. **Functional correctness — the gold standard.** Where the output can be *executed or checked*,
   do that: run the generated code against tests, verify the math, validate structured output
   against its schema, compare final system/database state to an annotated goal state. This is a
   real oracle. Prefer it always when available.
2. **Deterministic checks.** Format validity, schema conformance, constraint satisfaction,
   citation-exists checks, regex/rule assertions.
3. **Comparative (pairwise) evaluation.** For subjective quality, A-vs-B preference is more
   reliable than absolute scoring.
4. **LLM-as-judge.** Only for genuinely subjective output, and only once validated per §6.
5. **Lexical similarity (BLEU, ROUGE).** Use with extreme caution and never alone — a good
   response phrased differently scores badly. Semantic similarity via embeddings is better but
   inherits the embedding model's blind spots.

## 6. LLM-as-judge validation protocol (mandatory before any judge result is trusted)

An LLM judge inherits every failure of the model it judges. Documented biases: **position bias**
(favours the first-presented answer), **verbosity bias** (favours longer answers, errors included),
**self-enhancement bias** (favours its own generations — roughly 10% win-rate inflation), and
**non-reproducibility** (probabilistic scoring).

A judge is admissible only if all of the following hold:

- [ ] Validated against **human labels on held-out data** the judge prompt was not tuned on.
- [ ] Agreement reported as **precision and recall separately**, not raw agreement — raw agreement is
      misleading under class imbalance. Iterate the prompt until alignment converges (target >90%).
- [ ] **Temperature 0**, fixed prompt, pinned model version. Record all three.
- [ ] **Position randomised** across pairs; length controlled or explicitly measured as a confound.
- [ ] The judge is **not the same model** that produced the output, where avoidable. If it is,
      state that self-enhancement bias is uncontrolled and treat the result as an upper bound.

**A judge without measured precision/recall is an uncalibrated instrument.** Findings resting on
it are `Uncertain`. Flag an unvalidated judge as **High** severity in review.

## 7. RAG evaluation — decompose retrieval from generation

A single end-to-end score cannot tell you which half is broken. Report all four (RAGAS decomposition):

| Metric | Diagnoses | Failure it exposes |
|---|---|---|
| **Context precision** | Retrieval | Irrelevant chunks ranked highly |
| **Context recall** | Retrieval | Necessary evidence never retrieved |
| **Faithfulness** | Generation | Claims not inferable from context (hallucination) |
| **Answer relevance** | Generation | Off-topic, vague, or incomplete answers |

Add ranking metrics for the retriever (**NDCG**, **MRR**) and evaluate the retriever in isolation
as well as end-to-end. **Do not report the mean of the four as a headline number** — averaging
re-collapses precisely the distinction the decomposition exists to make.

## 8. Agent evaluation — reliability, not a single lucky run

- **Compounding error:** 95% per-step accuracy is ~60% task accuracy over 10 steps. A healthy
  per-step metric is fully consistent with a system that usually fails the task. Always report both.
- **pass^k (τ-bench):** the fraction of scenarios solved on **all k independent rollouts**. Unlike
  pass@k it *falls* as k rises, which is the point — it measures reliability. Reference result:
  GPT-4o scored under 50% at pass^1 but below 25% at pass^8.

**Requirements:**
- [ ] Report **pass^k with k ≥ 5** alongside pass^1 for any agentic system. Single-run success on a
      nondeterministic system is not evidence.
- [ ] Score the **trajectory**, not only the final answer: intermediate action correctness, tool
      selection distribution, policy compliance, recovery from malformed output.
- [ ] Where possible score by **final state vs. annotated goal state** rather than by a judge (§5.1).
- [ ] Track tool-usage patterns to surface underused, redundant, or misrouted tools.

## 9. Eval set curation

One test suite is not an evaluation. Curate **multiple sets**:

- **Production-distribution baseline** — what users actually send.
- **Sliced sets** — by user tier, region, language, input length, or any axis you claim to serve.
- **Known-failure-mode sets** — one per mode found in §3, plus typos, adversarial inputs, and
  out-of-scope queries.

Rules:
- *If you care about something, put a test set on it.*
- **Bootstrap to check sufficiency** — if results swing wildly across resamples, the set is too small.
- **Report wins per slice.** An aggregate improvement that reverses on a slice is not an improvement.
- **Drop perfectly correlated metrics** — keep one per correlated pair.
- **Held-out means held-out**: never trained on, never tuned on, never used for prompt iteration.
- **Contamination check**: for foundation models, verify the eval set did not leak through
  *pretraining*, not just through your own train/test split.

## 10. Statistical reporting (replaces "average the runs")

Averaging does not establish significance.

- State **n** (number of seeds/runs) explicitly. n=1 is an anecdote — label it as such.
- Report **dispersion**: stratified bootstrap confidence intervals and **interquartile mean (IQM)**
  rather than bare mean/median point estimates. `rliable` implements this for the small-n regime.
- Report **effect size**, not just direction.
- Comparisons must share data splits, preprocessing, hardware, stopping criteria, and seed count.
- **Flag as High severity** any comparison reported without dispersion, and any claim of
  significance supported only by a mean.

## 11. Observability (what to log)

*Log everything*, with traceable tags enabling step-by-step replay of any single query.

- **Performance:** TTFT, TPOT, throughput (TPS), cost per inference.
- **Quality & safety:** format-failure rate, factual consistency, toxicity, **false refusals**.
- **User engagement:** early termination, conversation turn count, regeneration requests — the
  cheapest honest real-world signal available, and the one most often missing.
- **Component health:** retriever hit rate and latency, tool-selection distribution, cache hit rate.
- **Agentic:** full per-step trace — prompt, tool call, tool result, state delta.

## 12. Offline → online

Offline evaluation is a proxy. Close the loop:

- **Map each metric to a business outcome.** "What does 80% factual consistency mean for this
  product?" If nobody can answer, the metric is decorative.
- **Rollout plan required** before serving: shadow deployment or A/B, guardrail metrics monitored
  live, and an explicit **rollback trigger** stated as a threshold.
- **Production feedback loop:** sampled production traces flow back into §3 error analysis on a
  stated cadence. Evaluation is continuous, not a stage that completes.

---

## Evaluation Spec Block

Every blueprint STAGE 2.5 and every evaluation review must produce this block:

```
EVALUATION SPEC
- Task type: <classification | generation | retrieval | agentic | other>
- Primary metric: <name> on <eval set name>
- Measurement method: <functional correctness | deterministic | comparative | judge> (§5 tier)
- Baseline: <system> scoring <value>
- Minimum improvement that counts: <effect size>
- Guardrail metrics (must not regress): <list with thresholds>
- Kill criterion: <result at which this approach is abandoned>
- Eval sets: <production | slices | failure-mode sets — names and sizes>
- n (runs/seeds): <n>, dispersion reported as <bootstrap CI | IQM>
- Judge validated: <yes — precision X / recall Y on held-out human labels | no judge used | NO — result is Uncertain>
- Error analysis: <N traces reviewed, M failure modes, top mode = X at Y%>
- Observability: <which §11 dimensions are instrumented>
- Rollout & rollback: <mechanism, guardrail, trigger>
```
