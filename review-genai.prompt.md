# Generative AI Review Framework

Domain-specific review checklist for GenAI/LLM codebases. Loaded by the Code Review agent when the project involves LLMs, RAG, prompt engineering, or text generation.

## 1. Prompt & Context Management

- Inspect prompt construction quality and injection exposure.
- Review context window management (strategies for text chunking, summarization before hitting token limits).
- Evaluate retrieval pipeline (RAG) quality, chunking strategy, and grounding/citation strategy.
- Check for system prompt leakage in user-facing outputs.

## 2. Cost-Efficiency & Performance

- Identify unnecessary model calls or overly expensive models used for simple routing tasks.
- Check for excessive context stuffing, poor caching strategies, and repeated/redundant embeddings generation.
- Inspect lack of batching in generation APIs.
- Verify token counting is accurate before API calls.

## 3. Resilience & Generation Quality

- Check hallucination mitigation strategies (grounding, RAG, structured output).
- Verify fallback behaviors (what happens if the LLM provider API times out or throws a 529 error).
- Assess evaluation quality for generation tasks (e.g., LLM-as-a-judge prompts, RAGAS metrics).
- Check for proper streaming error handling if using SSE/streaming responses.
