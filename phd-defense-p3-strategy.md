# PhD Defense Agent — Phase 3: Narrative Strategy, Slide Structure & Figure Placement
# File: ~/.claude/prompts/phd-defense-p3-strategy.md
# Invoked after Phase 2 is approved. Read phd-defense-context.md first.

---

## PURPOSE

This phase has three sequential steps that must each be approved before the next begins:
1. Narrative strategy selection (with proof sketch placement per theorem)
2. Per-theorem slide budget negotiation
3. Full numbered slide outline generation with figure placement map

This phase produces the blueprint that Phase 4 (mockup) and Phase 5 (LaTeX) will execute.

---

## AUTHORITATIVE FOUNDATIONS

**Swath-and-Dive Pattern — Rodrigues-Belo, A. (2022):**
https://ahappyphd.org/posts/swath-dive/
"More often than not, too much time is spent in the introductory and related literature
parts of the presentation, which are somehow 'safe,' and time runs out when the student
is getting to the really interesting part — the student's own work."
The Swath-and-Dive pattern enforces a strict allocation:
- Swath (~50% of slides): overview of all key elements with equal weight
- Dive (~30% of slides): deepest zoom on the most impressive or novel contribution
- Limitations + Future Work + Conclusion (~20%): mandatory, not optional

**NPS Defense Outline — Rowe, N.:**
https://faculty.nps.edu/ncrowe/phd_defense_outline.htm
Provides the slide-by-slide technical breakdown and pacing guidance (1 slide/minute).

**Nancy Duarte — Resonate (Wiley, 2010):**
https://www.duarte.com/books
Sparkline theory: audiences navigate more confidently when they can see the shape of
the whole arc from the start. Motivates the Visual Roadmap slide and the Discussion
Callback technique.

**Matt Might — "Academic Martial Arts: Defending Your PhD":**
https://matt.might.net/articles/phd-defense-tips/
The contributions slide is the most important slide in the talk. State contributions
as claims early and clearly, and return to them in the conclusion.

---

## STEP 1 — NARRATIVE STRATEGY SELECTION

Present the user with three options. Explain trade-offs clearly. Do NOT choose on their
behalf. The proof sketch placement is asked as a separate sub-question after strategy
selection.

---

### Option A — Classical Linear (NPS / Swath-and-Dive Default)
**Order:** Motivation → Contributions → Related Work → Methods & Framework →
Main Results → [Proof Sketches — placement TBD] → Discussion → Limitations → Conclusion

**Best for:** Work where the method is the novel contribution and must be understood
before results are appreciated. The audience needs to see the machinery before the
payoff makes sense.

**Trade-off:** Risk of spending too much time in the Swath before reaching the Dive.
Requires strict time discipline.

---

### Option B — Results-First (High-Impact Opening)
**Order:** Motivation → Contributions → Main Results stated upfront →
Literature gap (why existing methods fail) → Methods & Proof Sketches
(how your results are achieved) → Discussion → Limitations → Conclusion

**Best for:** Work with a striking, counterintuitive, or clearly superior result that
hooks the audience immediately. The payoff is shown first; the machinery justifies it.

**Trade-off:** Requires that the results be stated at a level the broad audience can
appreciate before the technical framework is introduced. May need extra care on the
motivation-to-results transition.

---

### Option C — Gap-Driven (Literature-Forward)
**Order:** Motivation → Literature gap as the central driver → Contributions as the
direct answer to the gap → Methods → Main Results →
[Proof Sketches — placement TBD] → Discussion → Limitations → Conclusion

**Best for:** Work that is best motivated by showing the failure or incompleteness of
prior art first. The gap is the story; the contributions are the resolution.

**Trade-off:** Requires a compelling and precise characterization of the gap. If the
related work section is thin, this strategy weakens the motivation.

---

### Proof Sketch Placement (asked after strategy selection)
After the user chooses a strategy, ask separately:

*"For each main theorem, where do you want to place its proof sketch slide? Please
choose one of the following for each theorem listed below:*
*(a) Immediately after the theorem statement slide*
*(b) Grouped together after all main results are stated*
*(c) Integrated into the methods section as logical steps leading to the result"*

List each theorem from the Phase 1 inventory by label and one-line statement.
Collect the user's placement decision per theorem.

Write approved strategy and placement decisions to CLAUDE.md. Wait for explicit approval
before proceeding to Step 2.

---

## STEP 2 — PER-THEOREM SLIDE BUDGET

The most common Applied Mathematics defense failure is spending the entire Dive on the
first theorem and rushing or omitting the remaining contributions.

### Actions
From the Phase 1 inventory and the approved narrative strategy, propose a slide budget
for each main theorem:

| Theorem | Treatment level | Slides allocated | Rationale |
|---|---|---|---|
| Thm X.X | Full (statement + proof sketch + validation) | N | Central contribution |
| Thm X.X | Statement + brief sketch | N | Important but secondary |
| Thm X.X | Statement only | N | Supports main result |

**Treatment level definitions:**
- **Full treatment:** theorem statement slide + proof sketch slide (crux/trick) +
  validation/numerical evidence slide
- **Statement + brief sketch:** theorem statement slide + one slide showing the logical
  arrow chain only (no detailed crux)
- **Statement only:** theorem stated on a results summary slide, full proof in appendix

Present the proposed budget to the user with rationale. Ask:
*"Does this allocation reflect the relative importance of your contributions? Which
theorem should receive the deepest treatment in your Dive?"*

Wait for explicit approval before proceeding to Step 3.

---

## STEP 3 — FULL SLIDE OUTLINE GENERATION

Generate the complete numbered slide outline using the approved strategy, proof sketch
placements, and per-theorem budget.

### Mandatory Slide Elements
Every outline must include these elements in the approved narrative order:

| Element | Slide count | Zone | Notes |
|---|---|---|---|
| Title slide | 1 | — | `\titlepage` |
| One-Sentence Message | 1 | Swath | Verbatim approved sentence from Phase 2 |
| Visual Roadmap | 1 | Swath | Horizontal/vertical chapter timeline (Duarte Sparkline) |
| Motivation & Problem Statement | 2–3 | Swath | Hook works for non-expert; context→question arc |
| Contributions as Claims | 1–2 | Swath | Most important slide; contributions as precise claims |
| Related Work (NPS two subtypes) | 2–4 | Swath | Subtype 1: same problem; Subtype 2: same technique |
| Methods & Mathematical Framework | 5–10 | Swath | Definitions, assumptions, formulation, key machinery |
| Main Results + Proof Sketches | 3–8 | Dive | Per approved budget and placement decisions |
| Discussion (with Callback Visual) | 2–3 | Dive | Reuses exact visual from Motivation slide |
| Limitations & Future Work | 2–3 | Limits | Mandatory; specific not vague |
| Conclusion | 1–2 | — | Echoes contributions slide; one-sentence message verbatim |
| Backup slides | 5–10 | Appendix | In `\appendix`; ECHER categories |
| **Total main** | **~35–45** | | Calibrated for 45-minute talk |

### Output Format for Each Slide
```
[N] ZONE | ASSERTION TITLE (full sentence) | Content type | Source section | Est. time
```

**Content types:** theorem-statement / proof-sketch / figure-evidence /
equation-annotated / tikz-flowchart / roadmap / text-argument / table / placeholder

### Assertion-Evidence Title Validation
Before presenting the outline, check every slide title:
- Is it a **full sentence making a claim**? e.g. "Algorithm X converges in O(n log n)
  under Assumption A" ✓
- Is it a topic label? e.g. "Complexity Analysis" ✗ — rewrite as an assertion

Source: Michael Alley — *The Craft of Scientific Presentations*, 2nd ed. (Springer, 2013),
Chapter 4. https://link.springer.com/book/10.1007/978-1-4419-8279-7
"Instead of building presentations on the weak foundation of topic phrases and bulleted
lists, the assertion-evidence approach calls for building presentations on succinct
message assertions supported by visual evidence."

### Discussion Callback Flag
Flag the Discussion section slides with:
`% CALLBACK: Reuse exact visual from Motivation slide [slide N] — Duarte narrative arc closure`

### Backup Slides Structure
The appendix must cover the following ECHER question taxonomy categories:
- **Framing and motivation:** Why this title/framing? How did research questions evolve?
- **Literature:** Why these areas and not others? Which scholars were most influential?
- **Methodological:** Why these methods? What alternatives were considered?
- **Results and validity:** Most significant findings? Main challenges?
- **Forward-looking:** Where does this work go next? What if the approach had failed?

Source: ECHER database of PhD defense questions (echer.org/defense-questions/);
Prof. Jason H. Moore — *Inside Higher Ed* (March 2021).
https://www.insidehighered.com/advice/2021/03/12/question-phd-candidates-should-be-prepared-answer-their-dissertation-defense

### Figure Placement Map
After presenting the outline, propose which figure from the Phase 1 inventory goes on
which slide. Present as a table:

| Slide N | Assertion title (brief) | Figure filename | Status |
|---|---|---|---|
| 7 | [title] | convergence_plot.pdf | ✓ confirmed |
| 9 | [title] | comparison_table.png | ⚠ raster — request vector |
| 12 | [title] | PLACEHOLDER | % TODO: [exact description] |

Flag every slide needing a figure not in the folder with PLACEHOLDER status.

### Pacing Note
Target: 1 slide per minute. Structure so that 60–75% of main slides form a concise core
and the remaining 25–40% can be expanded or skipped depending on time.
Source: SciSpace (2025) — https://scispace.com/resources/phd-defense-tips/

---

## STEP 3 COMPLETION

Write the approved outline to `defense_outline.md` in the thesis folder.
Write figure placement map to CLAUDE.md under Phase 3 approved decisions.
Mark Phase 3 status as `approved` in CLAUDE.md.

Confirm: *"Outline and figure map approved. Writing to defense_outline.md.
Invoking Phase 4 — Content Mockup."*
