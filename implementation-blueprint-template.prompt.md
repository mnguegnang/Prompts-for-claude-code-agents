# Implementation Blueprint Template (PI Phase 4)

Structure for generating `[Topic_Name]_Implementation_Blueprint.md` — the exact instruction manual
for the Coder Agent.

**Before writing STAGE 2.5, load `~/.claude/prompts/evaluation-methodology.prompt.md`.** The
evaluation spec is the part of this document that defines what "done" means; everything else only
defines what gets built.

## Industrial LLMOps Lifecycle Structure

- **STAGE 1 — Environment & Strict Versioning:** Tech stack with strict version pinning for
  volatile libraries. Hardware optimization flags (precision, memory mapping, quantization). Use
  `mcp__academic-research__search_huggingface_hub` to find models matching `task`,
  `max_parameters`, and `required_quantization` constraints.

- **STAGE 2 — Data, State, & Context Pipeline:** RAG: chunking strategies, embedding models,
  vector DB schemas. Agentic: memory architecture, state management. Fine-tuning: tokenization,
  dataset splits. SECURITY: PII masking before data enters LLM context.
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
  A PoC that runs but is unmeasured has not completed this stage.

- **STAGE 4 — Core Implementation & Orchestration:** Agentic orchestration (ReAct, Plan-and-Solve),
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
  primary metric means for this product, the metric is decorative.

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
      considered for the relevant surfaces (LLM01/05/06/08/10 at minimum for RAG or agentic work).
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
