# Generative AI Review Framework

Domain-specific review checklist for GenAI/LLM codebases (LLMs, RAG, prompt engineering, text
generation). Loaded by the Code Review agent alongside `review-methodology.prompt.md` and
`evaluation-methodology.prompt.md`.

## 1. Security — OWASP Top 10 for LLM Applications 2026

Work the named taxonomy so findings are comparable and nothing is silently omitted. Report each
finding with its 2026 ID. Two framing rules from the 2026 edition govern every row [O1 p. 5–7, 13]:

- **Assume the model will be fooled; build the system so nothing important breaks.** No reliable
  prevention for prompt injection exists, so defences are judged by the blast radius they bound,
  not by how much injection they filter.
- **Component vs. actor.** This list covers the model as a component. Once it calls tools, keeps
  memory across sessions, or triggers downstream actions, also work `review-agentic.prompt.md`
  (OWASP Agentic Top 10).

| ID | Risk | What to check in this codebase |
|----|------|-------------------------------|
| **LLM01** | Prompt Injection | Direct **and indirect** — retrieved documents, tool and MCP output, emails, issue titles, persistent memory, and **images, audio, and video**. An unmitigated indirect path is **Critical**. Invisible Unicode (tag-block U+E0000–E007F, variation selectors U+FE00–FE0F, zero-width U+200B–D, U+2060) stripped at every ingest *and* render boundary. External content passed in a separate, provenance-labelled channel. **Rule of Two:** any component with untrusted input + sensitive data + state change or external communication needs per-action human approval showing the exact action. Defence claims tested against an adaptive attacker who has read the defence, not a static attack set. [O1 §LLM01 mitigations 3–8, 11] |
| **LLM02** | Sensitive Information Disclosure | PII entering context unmasked; secrets in prompts; context echoed to users. **Authorise before retrieval**, not after. Reasoning traces classified and redacted as output. Logs and traces scrubbed before APM ingestion; no-train/no-retain enforced technically, not by policy text. Log-probs and confidence gated on production endpoints; per-user query budgets on sensitive endpoints. [O1 §LLM02 mitigations 2–11] |
| **LLM03** | Excessive Agency *(was LLM06)* | Minimum tools, functionality, and permissions; **no open-ended tools** (run shell, fetch any URL) where a narrow one works. Tools execute in the requesting user's security context. **Complete mediation:** authorisation decided in code, never by the LLM. Human approval for high-impact actions; circuit breakers on tool-invocation thresholds. [O1 §LLM03 mitigations 1–9] |
| **LLM04** | Supply Chain *(was LLM03)* | Unpinned model versions; `trust_remote_code=True`; unaudited dependencies (`pip-audit`). Models, adapters, and datasets listed in an AI-BOM / ML-SBOM; artifacts hash-pinned and verified at **promotion** — model conversion and merge services are high-risk promotion points. MCP and tool packages per `review-agentic` §1. [O1 §LLM04 mitigations 3–5] |
| **LLM05** | Data & Model Poisoning *(was LLM04)* | Untrusted documents written into the vector store; user content entering fine-tuning data unreviewed; no provenance for indexed corpora. **Inference artifacts are code:** chat templates, tokenizer configs, LoRA/PEFT adapters, and quantisation files are reviewed and pinned. Datasets under version control for rollback. Retraining and preference-feedback loops validated and rate-limited. Safety alignment does not remove backdoors — probe for triggers. [O1 §LLM05 mitigations 7–11] |
| **LLM06** | Unbounded Consumption *(was LLM10)* | Hard, non-overridable spend caps per key, user, and account; agent runs bounded by **step, recursion-depth, time, and per-run cost** limits; queued-action limits; serving framework kept patched. An approved cost *estimate* is not a control — look for the runtime guard. [O1 §LLM06 mitigations 1, 5, 8, 9] |
| **LLM07** | Misinformation *(was LLM09)* | Now a system-level risk: a wrong answer becomes a wrong action. **Claim-check-act** — generation separated from execution, claims verified against authoritative, current sources before acting. Tool calls checked for arguments, authorisation, preconditions, and current state. Structured outputs with mandatory fields to catch omissions. Groundedness and consistency checks, not model confidence. [O1 §LLM07 mitigations 1–6] |
| **LLM08** | Hidden Context Exposure *(was System Prompt Leakage, LLM07)* | Hidden context = system prompt, developer instructions, retrieved policy text, and **tool/function schemas**. Assume all of it is discoverable: no credentials or connection strings in it, and it is never the sole boundary for authorisation, privilege separation, or content filtering — those run deterministically in code. Keep a test asserting it is not echoed. [O1 §LLM08 description, mitigations 1–2] |
| **LLM09** | Vector & Embedding Weaknesses *(was LLM08)* | Multi-tenant stores without per-tenant filtering; no access control on the index. **Mixed-trust content never shares an index** without hard isolation. Content normalised before embedding; ingest-time anomaly detection for vectors that sit close to many queries (retrieval hijacking). Embeddings deleted when the source is, with reconciliation audits; immutable retrieval logs. Every blueprint here specifies a vector DB — this is never not applicable. [O1 §LLM09 mitigations 1–5] |
| **LLM10** | Improper Output Handling *(was LLM05)* | Model output reaching `eval`, a shell, SQL, a file path, or HTML without validation; structured output not schema-validated. Context-aware output encoding; parameterised queries; CSP. ANSI escapes and control characters stripped before terminals or logs. **Renderers never auto-fetch** model-referenced images or link previews (exfiltration channel). Now also covers insecure code generated by assistants. [O1 §LLM10 examples 6–7, mitigations 2–8] |

Reports written against the 2025 list: LLM03→04, LLM04→05, LLM05→10, LLM06→03, LLM07→08,
LLM08→09, LLM09→07, LLM10→06; LLM01 and LLM02 unchanged.

**Source:** [O1] OWASP GenAI Security Project, *OWASP Top 10 for LLM Applications 2026* (published
2026-08-03) — https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/ (PDF:
https://genai.owasp.org/download/56857/). Rationale pp. 5–7; entries pp. 10–57.

## 2. Prompt & Context Management

- Prompt construction quality; separation of instruction from interpolated data.
- Context-window management: chunking, summarisation, truncation strategy at the limit.
- Retrieval pipeline quality: chunking strategy, overlap, metadata, reranking.
- Work `llm-context-grounding.prompt.md` §1–5, §7 as a checklist: full-context baseline beaten;
  representation fits the query shape; hybrid BM25+dense with per-stage retrieval@k gains;
  evidence position and the position slice; cache-friendly prompt order; hash-keyed ingestion with
  a pinned embedding model.
- Grounding and citation strategy — can a claim be traced to a retrieved span?

## 3. Cost-Efficiency & Performance

- Unnecessary model calls; an expensive model used for a simple routing task.
- Excessive context stuffing; missing prompt caching; repeated/redundant embedding generation.
- Missing batching in generation APIs.
- Accurate token counting before calls; enforced budget guard (see LLM06).
- Optimisation order and re-gating: `production-operations.prompt.md` §3.

## 4. Resilience

- Hallucination mitigation: grounding, RAG, structured output, refusal paths.
- Fallback behaviour on provider timeout, 429, or 529 — is there a secondary model or a graceful
  degradation path?
- Streaming/SSE error handling mid-stream.
- Retry logic with exponential backoff **and** a spend ceiling.

## 5. Evaluation — the part most GenAI projects get wrong

Load `evaluation-methodology.prompt.md` and apply it. In this domain specifically:

### 5.1 RAG must be decomposed (§7 there)

A single end-to-end score cannot say which half is broken. Require all four:

| Metric | Diagnoses | Exposes |
|---|---|---|
| Context precision | Retrieval | Irrelevant chunks ranked highly |
| Context recall | Retrieval | Necessary evidence never retrieved |
| Faithfulness | Generation | Claims not inferable from context |
| Answer relevance | Generation | Off-topic or incomplete answers |

Plus **NDCG/MRR** for the retriever evaluated in isolation. Flag a project reporting only an
end-to-end score, or only the mean of the four, as **High** — the aggregate re-collapses exactly
the distinction that makes failures attributable.

### 5.2 Any LLM judge must be validated (§6 there)

Check for: precision/recall against held-out human labels, temperature 0, pinned model version,
randomised position, length confound controlled, and judge ≠ generator where avoidable. Position,
verbosity, and self-enhancement bias are documented and material (self-bias inflates win rate by
roughly 10%). **An unvalidated judge is High severity and its metrics are not evidence.**

### 5.3 Method choice

Prefer functional correctness wherever the output can be executed, checked, or schema-validated.
Reserve judges for genuinely subjective output. Flag BLEU/ROUGE used as a primary quality metric.

### 5.4 Error analysis

Confirm someone has actually read ~30 traces and built a failure taxonomy. If not, do it as part
of the review and report the frequency table — it will reorder the findings.
