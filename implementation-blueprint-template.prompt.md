# Implementation Blueprint Template (PI Phase 4)

Structure for generating `[Topic_Name]_Implementation_Blueprint.md` — the exact instruction manual
for the Coder Agent.

**Before writing STAGE 2.5, load `~/.claude/prompts/evaluation-methodology.prompt.md`.** The
evaluation spec is the part of this document that defines what "done" means; everything else only
defines what gets built.

**For any LLM, RAG, or agentic system, also load `~/.claude/prompts/llm-context-grounding.prompt.md`**
(STAGES 1, 2, 4) and, for anything that will be served, `~/.claude/prompts/production-operations.prompt.md`
(STAGE 6).

## Industrial LLMOps Lifecycle Structure

- **STAGE 0 — Product Framing (before anything technical):** the users and their problem; why AI
  beats a rule- or heuristic-based solution here (often it does not); non-goals; which error costs
  more, false positive or false negative; and the project stage — prototype, MVP, or enterprise —
  which sets how much of the rest of this template applies. If asked to build without a spec,
  write one. [B1 §Making product decisions, §Driving the build loop; B2 §Decide if AI adds unique value]

- **STAGE 1 — Environment & Strict Versioning:** Tech stack with strict version pinning for
  volatile libraries. Hardware optimization flags (precision, memory mapping, quantization). Use
  `mcp__academic-research__search_huggingface_hub` to find models matching `task`,
  `max_parameters`, and `required_quantization` constraints. Model tiers, reasoning effort, and
  knowledge cutoff per `llm-context-grounding` §6.

- **STAGE 2 — Data, State, & Context Pipeline:** RAG: chunking strategies, embedding models,
  vector DB schemas. Agentic: memory architecture, state management. Fine-tuning: tokenization,
  dataset splits. SECURITY: PII masking before data enters LLM context. **Context decisions**
  (full-context baseline, up-front vs. just-in-time per source, representation, ingestion) per
  `llm-context-grounding` §1–2, §7. **Trained models:** one feature code path for training and
  serving (`production-operations` §1).
  **Provenance & licensing (mandatory):** every dataset and model records source, licence, and
  whether that licence permits the stated deployment. A non-commercial asset in a production
  build, or an asset of unknown provenance, must be flagged here — not discovered at review.

- **STAGE 2.5 — Evaluation Specification (written BEFORE anything is built):**
  Evaluation-Driven Development: define the evaluation before building the thing it evaluates.
  A system built first and measured afterwards has no definition of failure, so it cannot fail.
  This stage produces the **EVALUATION SPEC block** from `evaluation-methodology.prompt.md`,
  containing:
  - Capability buckets to be measured: domain-specific, generation, instruction-following, cost/latency.
  - **Pre-registered acceptance criteria** — baseline to beat, primary metric, minimum improvement
    that counts, guardrail metrics that must not regress, and the kill criterion at which this
    approach is abandoned rather than tuned. **Thresholds are frozen once implementation starts;**
    moving them after seeing results is a logged deviation requiring justification.
  - Measurement method per §5 preference order (functional correctness > deterministic checks >
    comparative > judge), and the judge-validation plan if a judge is used at all.
  - Eval sets: production-distribution baseline, slices, and known-failure-mode sets.
  - n (seeds/runs) and how dispersion will be reported.
  - The criteria taxonomy is revisable from error analysis; the thresholds are not.

- **STAGE 3 — Baseline PoC:** the simplest baseline (zero-shot or naive retrieval), whose purpose
  is to **produce the first number on the STAGE 2.5 eval set** — not merely to prove connectivity.
  A PoC that runs but is unmeasured has not completed this stage. Where the task allows, also
  score a **heuristic/rule baseline and a simple classical model**, and report every candidate
  with accuracy, training time, and inference latency/cost. A simpler model that ties is better. [B3 #1, #4; B4 Model 5]

- **STAGE 4 — Core Implementation & Orchestration:** **Justify the architecture rung by rung:**
  single tuned LLM call (with retrieval and examples) → fixed workflow → agent loop → multi-agent.
  Each step up needs a measured gap on the rung below; multi-agent costs ~15× the tokens of chat.
  Use deterministic code for any step whose output is computable. [B5 §When (and when not) to use
  agents; B6] For agents, name the prompt-injection containment pattern (`review-agentic` §1). Agentic orchestration (ReAct, Plan-and-Solve),
  tool definitions with least-privilege scoping, LoRA parameters, decoding strategies, API retry
  logic (exponential backoff), fallback model architecture. **Enforced budget guard**: every paid
  API path and agent loop carries `max_iterations` and `max_spend` read from config.

- **STAGE 5 — Guardrailed Evaluation, Error Analysis & Logging:** Execute the STAGE 2.5 spec.
  Mandatory error-analysis pass (~30 traces, binary verdict + critique, failure taxonomy,
  frequency table) driving the fix priority. Trajectory evaluation and **pass^k (k ≥ 5)** for
  agentic systems; RAGAS four-way decomposition for RAG. Observability per §11: TTFT/TPOT/TPS,
  cost per inference, format-failure rate, false refusals, user-engagement signals, per-step
  traces. Output guardrails. For governance framing on GenAI-specific risk, map to the NIST AI
  600-1 Generative AI Profile categories where the project is enterprise-facing.

- **STAGE 6 — CI/CD, Serving & Rollout:** Serving engine (vLLM, FastAPI), containerization
  (Docker), continuous delivery packaging. **Rollout plan is mandatory**, not optional: shadow
  deployment or A/B mechanism, live guardrail metrics, an explicit **rollback trigger** stated as
  a threshold, and the cadence at which sampled production traces flow back into error analysis.
  Each offline metric is mapped to a business outcome — if nobody can state what 80% on the
  primary metric means for this product, the metric is decorative. Drift detection, the disengage
  mechanism and its owner, and the cost-optimisation order: `production-operations` §1–3.

## Blueprint Self-Assessment Checklist

Run this against your own output before presenting the draft. Include the results in chat.

**Note on this checklist:** it is graded by its author, so it establishes completeness, never
correctness. Present it as such — it is a list of what you remembered, not evidence that any of
it is right.

- [ ] **Version Pinning:** All volatile libraries have exact versions specified.
- [ ] **Hardware Compliance:** Model selection and precision settings match Phase 1 compute constraints.
- [ ] **Provenance & Licensing:** Every dataset and model has a recorded source and licence, and
      that licence permits the stated deployment.
- [ ] **Security:** PII masking defined; least-privilege tool scoping; OWASP LLM Top 10 risks
      considered for the relevant surfaces (LLM01/03/06/09/10 (2026 IDs) at minimum for RAG or agentic work).
- [ ] **Evaluation Spec (STAGE 2.5):** EVALUATION SPEC block complete, written before STAGE 3.
- [ ] **Pre-registered Acceptance Criteria:** baseline, primary metric, minimum improvement,
      guardrails, and kill criterion are all stated with numbers.
- [ ] **Measurement Validity:** method chosen per the §5 preference order; judge-validation plan
      present if a judge is used.
- [ ] **Error Analysis Planned:** trace sampling and failure-taxonomy step specified in STAGE 5.
- [ ] **Statistical Plan:** n stated; dispersion method (bootstrap CI / IQM) stated.
- [ ] **Fallback Architecture:** backup model or strategy defined for primary failure.
- [ ] **Budget Guard:** enforced token/spend ceiling specified for every paid path and agent loop.
- [ ] **Reproducibility:** seeds, determinism flags, dataset versioning, checkpoint strategy.
- [ ] **Observability:** the §11 dimensions this project will instrument.
- [ ] **Serving, Rollout & Rollback:** serving engine, rollout mechanism, and rollback trigger.
- [ ] **Artifact Contact:** every field, key, column, and count named in the blueprint was checked
      against the released file (file + command cited), or is marked `ASSUMPTION — verify at load`.
      No data example is reconstructed from a paper figure.
- [ ] **Silent Failure Modes:** every data-transformation rule states the failure that raises no
      error and the test that catches it; joins use exact keys, or the heuristic's selected
      records were enumerated and inspected.
- [ ] **Gate Placement:** every gate lists its entry dependency and downstream dependents, sits at
      the earliest point its entry dependency holds, and names a repository file as its evidence.
- [ ] **Objective Restated:** the head of the blueprint states what it optimises for, in the
      user's words, and every trade-off names the property traded.
- [ ] **Product Framing (STAGE 0):** users, why AI over rules, non-goals, costlier error type, stage.
- [ ] **Simplest Sufficient Design:** non-LLM baselines scored (STAGE 3); each architecture rung
      above a single call justified by a measured gap (STAGE 4).
- [ ] **Power & Risk Tiers:** eval set sized by power analysis; each failure mode has a risk tier.

Mark each ✅ or ❌. For any ❌, explain what is missing and whether it is intentional (e.g. serving
not needed for a research prototype).

## Review & Adjustment Loop

1. Save as draft with the checklist results. Ask the user to review and request adjustments.
2. **Surface the risk, not just the plan.** Present the two or three assumptions on which the
   blueprint most depends and which would be most expensive to discover wrong at STAGE 5, and ask
   the user which they want proven early — at STAGE 3, on the smallest possible test. A phase gate
   that only collects preferences does not use the one reviewer in the loop who is not a model.
3. Only upon explicit user approval, output: *"The Blueprint is finalized. You may now invoke the
   ml-genai-coder agent to begin building. My research is complete."*

## Sources (for rules marked [B#])

| ID | Source | Where the argument is |
|---|---|---|
| B1 | Ng, AI Engineering Skills Map: Shaping the build — https://www.linkedin.com/pulse/ai-engineering-skills-map-shaping-build-andrew-ng-td7uc | §Driving the build loop; §Making product decisions |
| B2 | Google PAIR Guidebook, User Needs — https://pair.withgoogle.com/chapter/user-needs/ | §Decide if AI adds unique value; §Consider precision & recall tradeoffs |
| B3 | Google, Rules of Machine Learning — https://developers.google.com/machine-learning/guides/rules-of-ml | Rules #1, #4 |
| B4 | Breck et al., The ML Test Score — https://research.google.com/pubs/archive/aad9f93b86b7addfea4c419b9100c6cdd26cacea.pdf | Model 5 |
| B5 | Anthropic, Building effective agents — https://www.anthropic.com/engineering/building-effective-agents | §When (and when not) to use agents |
| B6 | Anthropic, How we built our multi-agent research system — https://www.anthropic.com/engineering/multi-agent-research-system | ~15× token cost; parallelisability of coding tasks |
