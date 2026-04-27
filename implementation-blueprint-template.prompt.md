# Implementation Blueprint Template (PI Phase 4)

Structure for generating `[Topic_Name]_Implementation_Blueprint.md` — the exact instruction manual for the Coder Agent.

## Industrial LLMOps Lifecycle Structure

- **STAGE 1 — Environment & Strict Versioning:** Tech stack with strict version pinning for volatile libraries. Hardware optimization flags (precision, memory mapping, quantization). Use `mcp__academic-research__search_huggingface_hub` to find models matching `task`, `max_parameters`, and `required_quantization` constraints.
- **STAGE 2 — Data, State, & Context Pipeline:** RAG: chunking strategies, embedding models, vector DB schemas. Agentic AI: memory architecture, state management. Fine-Tuning: tokenization, dataset splits. SECURITY: PII masking before data enters LLM context.
- **STAGE 3 — Baseline PoC:** Simplest baseline (zero-shot or naive retrieval) to prove connectivity.
- **STAGE 4 — Core Implementation & Orchestration:** Agentic orchestration (ReAct, Plan-and-Solve), tool definitions, LoRA parameters, decoding strategies, API retry logic (exponential backoff), fallback model architecture.
- **STAGE 5 — Guardrailed Evaluation & Logging:** Evaluation protocol (Trajectory Evaluation, RAGAS), experiment tracking, output guardrails.
- **STAGE 6 — CI/CD & Optimized Serving:** Serving engine (vLLM, FastAPI), continuous delivery packaging (Docker).

## Blueprint Self-Assessment Checklist

Before presenting the draft to the user, run this checklist against your own output. Include the results in the chat so the user knows what to verify:

- [ ] **Version Pinning:** All volatile libraries have exact versions specified.
- [ ] **Hardware Compliance:** Model selection and precision settings match Phase 1 compute constraints.
- [ ] **Security:** PII masking or data sanitization steps defined for data entering LLM context.
- [ ] **Evaluation Protocol:** Metrics, baselines, and evaluation cadence specified.
- [ ] **Fallback Architecture:** Backup model or strategy defined for primary failure.
- [ ] **Reproducibility:** Random seeds, dataset versioning, and checkpoint strategy specified.
- [ ] **Serving & Packaging:** Serving engine and containerization defined (if applicable).

Mark each item ✅ or ❌. For any ❌, explain what is missing and whether it is intentional (e.g., serving not needed for a research prototype).

## Review & Adjustment Loop

1. Save as draft with the checklist results. Ask the user to review and request adjustments.
2. Only upon explicit user approval, output: *"The Blueprint is finalized. You may now invoke the ml-genai-coder agent to begin building. My research is complete."*
