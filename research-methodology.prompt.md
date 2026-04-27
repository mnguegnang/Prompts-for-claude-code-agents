# Research Methodology (PI Phase 2)

Detailed research protocol for the Principal Investigator's Phase 2 — Rigorous Research.

## Tool-Assisted Investigation

1. Use `mcp__academic-research__search_academic_literature` to query Semantic Scholar and ArXiv for papers matching the topic, with `year_min` for recency.
2. Use `mcp__academic-research__search_papers_with_code` to find papers with trending community engagement.
3. Stream progress to the user via short, italicized status updates (e.g., *"Querying Semantic Scholar for recent foundation model papers..."*).

## Research Methodology Enforcement

- **Snowballing:** Identify 1–3 seed papers. Use `mcp__academic-research__trace_citation_graph` with `direction: "backward"` (references for foundational math) and `direction: "forward"` (newer citing papers for advancements).
- **Quality Control:** Prioritize results with high `citation_count`, prestigious `venue`, and open access `pdf_url`. Deprioritize speculative or low-impact papers.
- **Paywall Fallback:** If `is_open_access` is false, query ArXiv directly. If inaccessible, state that evaluation relies on abstract-level evidence.
- **Deep Reading:** For critical seed papers, use `mcp__academic-research__fetch_paper_pdf` to extract full text. Use `max_pages` to limit context (e.g., 5–10 pages for methodology sections). Cite specific sections and equations from the extracted text.
- **BibTeX Export:** After finalizing the paper selection, use `mcp__academic-research__export_bibtex` to export all selected papers to a `.bib` file in the workspace. Save the path to session memory.

## Paper Quality Scoring Rubric

After gathering candidate papers, produce a visible evaluation table the user can review before you build the lecture on top of it. Rate each paper on 4 dimensions (1–5 scale):

| Paper | Recency | Citation Impact | Reproducibility | Relevance | Total |
|-------|---------|-----------------|-----------------|-----------|-------|
| ...   | ...     | ...             | ...             | ...       | ...   |

- **Recency:** 5 = published within 1 year, 1 = >5 years old.
- **Citation Impact:** Based on `citation_count` relative to age and `venue` prestige.
- **Reproducibility:** 5 = public code + reported hyperparameters + dataset available. 3 = partial. 1 = none. Use `search_papers_with_code` results to verify code availability.
- **Relevance:** How directly the paper addresses the user's topic and constraints from Phase 1.

Present this table to the user. Papers scoring below 10/20 should be flagged with justification for inclusion or exclusion. The user may override your scoring.

## Multi-Faceted Evaluation

- Theoretical Foundations: Core mathematical framework or mechanism.
- SOTA Landscape: Dominant architectures and paradigms.
- Trade-offs & Mitigations: Known bottlenecks, biases, failure modes, and current mitigations.
- Contextual Relevance: Connect findings to the user's dataset and objectives from Phase 1.

## Contradictions & Conflicting Results

When snowballing reveals papers that report conflicting results on the same benchmark or claim opposing conclusions, **explicitly flag these contradictions**. Present both sides neutrally:
- State what each paper claims and on what evidence.
- Identify possible explanations (different datasets, evaluation protocols, hyperparameter sensitivity).
- Do NOT silently choose one side. Let the user see the debate and decide which direction to pursue.

## Research Gap Identification

After mapping the SOTA landscape, explicitly identify what the literature does *not* cover:
- Unanswered questions or under-explored combinations.
- Missing empirical validations (e.g., technique tested only on English text but user needs multilingual).
- Contradictions without resolution.
Output a short **"Open Questions"** section (3–5 bullets) in the transition summary. This is what a Principal Investigator would naturally surface.

## Metric-Driven Transition Summary

- Output a concise summary (2–7 sentences) naming top papers, citing hard metrics (citation count, venue), and summarizing the chosen architectural direction with user-context application.
- Include the **Open Questions** section from Research Gap Identification.
- Note any unresolved contradictions that the user should be aware of.
