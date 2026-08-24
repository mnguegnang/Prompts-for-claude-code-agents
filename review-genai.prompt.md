# Generative AI Review Framework

Domain-specific review checklist for GenAI/LLM codebases (LLMs, RAG, prompt engineering, text
generation). Loaded by the Code Review agent alongside `review-methodology.prompt.md` and
`evaluation-methodology.prompt.md`.

## 1. Security — OWASP Top 10 for LLM Applications (2025, v2.0)

Work the named taxonomy so findings are comparable and nothing is silently omitted. Report each
finding with its LLM ID.

| ID | Risk | What to check in this codebase |
|----|------|-------------------------------|
| **LLM01** | Prompt Injection | Direct **and indirect**: instructions embedded in retrieved documents, tool results, scraped pages, user-uploaded files. Is there any separation between instruction and data channels? Indirect injection through retrieved content is *the* defining RAG threat — treat an unmitigated path as **Critical**. |
| **LLM02** | Sensitive Information Disclosure | PII entering model context unmasked; secrets in prompts; training data or context echoed to users; logs capturing full prompts with user data. |
| **LLM03** | Supply Chain | Unpinned model versions; models/adapters/datasets from unverified sources; `trust_remote_code=True`; unaudited dependencies (run `pip-audit`). |
| **LLM04** | Data & Model Poisoning | Untrusted documents written into the vector store; user content admitted into fine-tuning data without review; no provenance record for indexed corpora. |
| **LLM05** | Improper Output Handling | Model output passed to `eval`, a shell, SQL, a file path, or rendered as HTML without validation. Structured output not schema-validated before use. |
| **LLM06** | Excessive Agency | Tools with more permission than the task needs; write/delete/spend capability without a human gate; no scoping of credentials per tool. |
| **LLM07** | System Prompt Leakage | Secrets or business rules embedded in the system prompt; prompt recoverable via user input; no test asserting it is not echoed. |
| **LLM08** | Vector & Embedding Weaknesses | Multi-tenant vector stores without per-tenant filtering (cross-tenant retrieval); embedding inversion exposure; no access control on the index; stale or contradictory entries never evicted. Every blueprint here specifies a vector DB — this is never not applicable. |
| **LLM09** | Misinformation | No grounding or citation strategy; ungrounded claims presented as fact; overreliance patterns with no confidence signalling to the user. |
| **LLM10** | Unbounded Consumption | No token/spend ceiling enforced in code; no rate limiting; unbounded retries; unbounded context growth. An approved cost *estimate* is not a control — look for the runtime guard. |

## 2. Prompt & Context Management

- Prompt construction quality; separation of instruction from interpolated data.
- Context-window management: chunking, summarisation, truncation strategy at the limit.
- Retrieval pipeline quality: chunking strategy, overlap, metadata, reranking.
- Grounding and citation strategy — can a claim be traced to a retrieved span?

## 3. Cost-Efficiency & Performance

- Unnecessary model calls; an expensive model used for a simple routing task.
- Excessive context stuffing; missing prompt caching; repeated/redundant embedding generation.
- Missing batching in generation APIs.
- Accurate token counting before calls; enforced budget guard (see LLM10).

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
