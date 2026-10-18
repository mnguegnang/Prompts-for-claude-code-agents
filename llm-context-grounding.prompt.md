# LLM Context, Grounding & Model Selection (shared — PI STAGE 1/2/4, Coder Step 3, Reviewer)

How an LLM system gets its context, which model runs it, and how documents become model input.
Covers Ng's skills-map areas *LLM foundations* and *Grounding models with data* [S0]. Load it when
specifying (PI), building (Coder), or reviewing (Reviewer, alongside `review-genai`) any LLM, RAG,
or agentic system. Each rule cites its source in the table at the end.

## 1. Decide where context comes from — before designing RAG

- **Full-context baseline first.** If the corpus fits in the prompt (roughly <200k tokens), the
  baseline is to include all of it with no retrieval. RAG must beat that baseline on the STAGE 2.5
  set to earn its complexity. [S1]
- **Per data source, choose: loaded up front or fetched just in time.** Just-in-time means the
  agent holds lightweight identifiers (file paths, stored queries, URLs) and loads content through
  tools at runtime; a hybrid loads a small stable core up front. Record the decision per source in
  the blueprint, with the reason. [S2 §Context retrieval and agentic search]

## 2. Match the data representation to the questions asked

Justify the choice against a sample of real user queries, not against the corpus:

| Query shape | Representation |
|---|---|
| Local lookup ("what does clause 4 say") | Vector / hybrid index |
| Global, corpus-wide ("what are the main themes") | Graph index: entity graph plus community summaries [S3] |
| Aggregates over structured records | Semantic layer of governed metrics, not text-to-SQL on raw tables [S4] |

## 3. Retrieval quality

- Pair dense embeddings with **BM25** so exact identifiers (error codes, SKUs, IDs) are not lost.
- **Contextualise chunks** (prepend chunk-specific context) before indexing.
- Add each stage (BM25, contextualisation, reranker) separately and measure its retrieval@k gain
  on the STAGE 2.5 set; a stage with no measured gain is removed. [S1 §BM25, §results]

## 4. Context is a budget

- Every extra token has diminishing, eventually negative, returns: aim for the smallest set of
  high-signal tokens. [S2 §Why context engineering is important]
- Put the highest-value evidence at the **start or end** of the context, never buried in the
  middle; add an eval slice that varies the position and amount of the gold evidence. [S5]
- **Long-running agents** specify: a compaction policy with an explicit list of what must survive
  it, clearing of stale tool results, and structured notes kept outside the window. [S2 §Compaction,
  §Structured note-taking]
- **Tool responses** are paginated, filtered, or truncated by default, and their error messages
  tell the agent what to do next. Tools are measured with an eval like any other component. [S6]

## 5. Prompt caching

Order every prompt: tools → system → static context and examples → per-request content. Nothing
volatile (timestamps, request IDs) before the cache breakpoint. Add a test asserting
`cache_read_input_tokens > 0` on the second identical-prefix call. [S7 §Structuring your prompt]

## 6. Choosing the lever and the model

- **Diagnose before choosing a lever.** Missing, outdated, or proprietary knowledge → context
  (retrieval, tools). Inconsistent format, tone, or reasoning → prompt, then fine-tuning. The
  levers solve different problems; there is no fixed prompting → RAG → fine-tuning ladder. [S8]
- **Benchmark at least two model tiers and the reasoning-effort levels** on the STAGE 2.5 set and
  choose from the cost–quality frontier. Consider a planner/doer split or a cascade (cheap model
  first, escalate on low confidence). [S9, S10]
- **Record each model's reliable knowledge cutoff.** Inject the current date and fresh facts
  wherever answers depend on recent information. [S11]

## 7. Ingestion is a tested pipeline

- **Choose the parse mode per document type.** Text-only extraction drops charts, tables, and
  layout; page-as-image keeps them at roughly 7× the tokens. Spot-check parsed output against the
  source document before indexing. [S12 §Document processing modes]
- **Key every document by ID plus content hash**: changed documents are re-embedded and upserted,
  unchanged ones skipped. Pin the embedding model version alongside the index; changing it forces a
  full re-embed. [S13 §Document Management]

## Sources

| ID | Source | Where the argument is |
|---|---|---|
| S0 | Ng, AI Engineering Skills Map: Building and deploying AI applications — https://www.linkedin.com/pulse/ai-engineering-skills-map-building-deploying-applications-andrew-ng-gyn5e/ | ¶ "LLM foundations", ¶ "Grounding models with data" |
| S1 | Anthropic, Contextual Retrieval — https://www.anthropic.com/news/contextual-retrieval | Intro (200k-token rule); §BM25; §results (49% fewer top-20 failures) |
| S2 | Anthropic, Effective context engineering for AI agents — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents | §Why context engineering is important; §Context retrieval and agentic search; §Compaction; §Structured note-taking |
| S3 | Edge et al., From Local to Global: A GraphRAG Approach — https://arxiv.org/abs/2404.16130 | Abstract |
| S4 | dbt Semantic Layer — https://docs.getdbt.com/docs/use-dbt-semantic-layer/dbt-sl | Intro |
| S5 | Liu et al., Lost in the Middle — https://arxiv.org/abs/2307.03172 | Abstract |
| S6 | Anthropic, Writing effective tools for agents — https://www.anthropic.com/engineering/writing-tools-for-agents | §Optimizing tool responses for token efficiency |
| S7 | Claude docs, Prompt caching — https://docs.claude.com/en/docs/build-with-claude/prompt-caching | §Structuring your prompt; §How automatic prefix checking works |
| S8 | OpenAI, Optimizing LLM accuracy — https://platform.openai.com/docs/guides/optimizing-llm-accuracy | §LLM optimization context |
| S9 | Chen et al., FrugalGPT — https://arxiv.org/abs/2305.05176 | Abstract (cascade, up to 98% cost reduction) |
| S10 | OpenAI, Reasoning best practices — https://platform.openai.com/docs/guides/reasoning-best-practices | §Reasoning models vs. GPT models |
| S11 | Claude docs, Models overview — https://docs.claude.com/en/docs/about-claude/models/overview | "Reliable knowledge cutoff" field |
| S12 | Claude docs, PDF support — https://docs.claude.com/en/docs/build-with-claude/pdf-support | §Document processing modes |
| S13 | LlamaIndex, Ingestion pipeline — https://developers.llamaindex.ai/python/framework/module_guides/loading/ingestion_pipeline/ | §Document Management |
