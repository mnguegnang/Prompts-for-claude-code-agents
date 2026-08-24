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

## Paper Admissibility Gates (binary — not a scored rubric)

Do **not** score papers 1–5 across dimensions and sum them. Unanchored ordinal scales are not
commensurable, summing them invents precision that does not exist, and a threshold on the sum
(e.g. "10/20") is arbitrary. The same antipattern is why LLM evaluation practice moved to binary
judgments: a pass/fail verdict plus a written justification forces an explicit decision about what
actually matters, where a 3-vs-4 distinction never does.

Apply **independent binary gates** and record a one-line justification per paper:

| Gate | Question | Pass condition |
|------|----------|----------------|
| **Provenance** | Is the source credible? | Peer-reviewed venue, OR preprint with substantive citation traction, OR authored by an established group in the area |
| **Relevance** | Does it address the user's topic *and* constraints from Phase 1? | Directly, not by analogy |
| **Reproducibility** | Could someone rebuild this? | Public code, reported hyperparameters, or an accessible dataset — state which of the three are present |
| **Evidence** | Are the claims supported? | Reports experimental conditions, baselines, and dispersion — not just headline numbers |
| **Currency** | Has it been superseded? | No later work invalidates the method for this use case |

Present the table with a **PASS/FAIL per gate** and the justification. A paper failing any gate may
still be included — but the inclusion must be argued explicitly, in writing.

**Do not score recency.** Penalising age structurally biases the survey toward novelty and
discards foundational mathematics, which is usually the part the user most needs grounded.
Recency is handled by the Currency gate (has this been superseded?), which is the question that
actually matters. Note publication year as metadata, not as a score.

The user may override any verdict.

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

## Reporting Evidence from Papers

When citing a result, carry its conditions with it. A benchmark number quoted without its
experimental setup is not transferable evidence.
- State the dataset, model scale, and hardware the result was obtained on.
- State **n** (seeds/runs) and dispersion if the paper reports them; state "not reported" if it
  does not — that absence is itself a finding about the paper's strength.
- Never present a single reported number as an expected outcome for the user's setting. Frame it
  as "reported under these conditions", and flag the gap to the user's constraints.
- Where papers disagree, keep both numbers and their conditions rather than averaging them.
