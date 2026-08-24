# PhD Defense Agent — Phase 4: Content Mockup
# File: ~/.claude/prompts/phd-defense-p4-mockup.md
# Invoked after Phase 3 is approved. Read phd-defense-context.md first.

---

## PURPOSE

Before any LaTeX is written, translate the approved outline into a text-based content
mockup — one entry per slide. This phase is the human checkpoint between planning and
execution. It catches misinterpretations of the thesis content, especially for proof
sketch slides, before they are encoded into LaTeX.

The mockup is not a draft of the slides. It is a structured description of what each
slide will contain and how its visual logic will work.

---

## AUTHORITATIVE FOUNDATIONS

**Michael Alley — The Craft of Scientific Presentations, 2nd ed. (Springer, 2013):**
https://link.springer.com/book/10.1007/978-1-4419-8279-7
Chapter 4 — Assertion-Evidence structure: every slide has a full-sentence assertion as
its title and visual evidence as its body. No bullet-point lists on core chapter slides.

**Paul Halmos — "How to Talk Mathematics" (1974):**
"Present the idea of the proof, not the proof itself." Every proof sketch slide must
capture the central insight — the trick — not the algebraic mechanics.

**John E. McCarthy — "How to Give a Good Colloquium" (AMS, 1999):**
"Show only the 'crux' of the argument or the main 'trick.' Omit technical lemmas
unless they are the central contribution."

**Jean-luc Doumont — Trees, Maps, and Theorems (Principiae, 2009):**
https://www.principiae.be
Present mathematical logic visually — as a flowchart of logical implications
(A ⇒ B ⇒ C) rather than as a vertical block of text and algebraic manipulation.

**Richard Mayer — Multimedia Learning, 2nd ed. (Cambridge UP, 2009):**
https://ctl.risepoint.com/principles-of-multimedia-learning/
Segmentation Principle: complex arguments must be broken into digestible steps.
Spatial Contiguity Principle: labels must be placed close to what they describe.

**Edward Tufte — The Visual Display of Quantitative Information, 2nd ed. (2001):**
https://washington.edu/brand/how-to-make-a-chart-tufte-style
Data-ink ratio: every figure mockup must describe a graph stripped of chartjunk.
Direct labeling on curves, not legend boxes.

---

## MOCKUP FORMAT

For every slide in the approved outline, produce one entry in this exact format:

```
=== SLIDE [N] ===
ZONE: [Swath | Dive | Limits | Appendix]
ASSERTION TITLE: [Full sentence claim — the headline of this slide]

BODY DESCRIPTION:
[Describe the visual evidence precisely. What does the audience see?
 Is it a figure, an annotated equation, a TikZ flowchart, a table?
 Describe it as if you are directing a visual designer.]

MATH CONTENT (if applicable):
[Write the key equation(s). Specify which terms will carry \underbrace or
 \overbrace annotations and what the annotation text will say.
 Example: \underbrace{Lf}_{\text{operator}} = \underbrace{\lambda f}_{\text{eigenvalue eq.}}]

PROOF LOGIC (for proof sketch slides only):
[Write the logical arrow chain from assumption to conclusion.
 Example: Assumption A (compactness) ⇒ Key Lemma 2.1 (uniform bound) ⇒ Theorem 3.2
 Identify explicitly: what is the central trick or non-obvious step?]

FIGURE:
[Filename from inventory, OR PLACEHOLDER with exact description:
 e.g. PLACEHOLDER: convergence curve comparing method A vs B on log scale,
      x-axis = iterations, y-axis = error norm, two curves labeled directly]

SPEAKER NOTES SKETCH:
  NARRATIVE: [1–3 sentence script — what to say on this slide]
  PACING:    [Any pause cue, blank-screen moment, or transition announcement]
  ANTICIPATE: [Most likely committee question triggered by this slide +
               one-line suggested response]
```

---

## SPECIAL RULES FOR PROOF SKETCH SLIDES

Proof sketch slides require extra care. Before describing the body, re-read the approved
Crux Statement from CLAUDE.md (Phase 2). The proof sketch slide must:

1. **Expose the trick.** The body must make the central non-obvious step visible —
   either as a TikZ arrow chain, a geometric diagram, or a single annotated key inequality.
   It must NOT be a compressed version of the full proof.

2. **Follow the Halmos/McCarthy rule.** If the proof sketch requires more than one
   display equation or more than 4 logical steps to describe, it is too detailed.
   Move the extra steps to a backup slide in the appendix.

3. **Identify the Doumont visual logic pathway.** For every proof sketch, write the
   logical arrow chain before anything else:
   ```
   [Starting assumption / known result]
   ⇒ [Key lemma or intermediate step — this is where the trick lives]
   ⇒ [Main theorem conclusion]
   ```
   This arrow chain IS the proof sketch slide content.

4. **Ask the user explicitly.** After presenting a proof sketch mockup, always ask:
   *"Does this mockup correctly identify the central trick of the proof? Is the arrow
   chain capturing the right logical pathway, or is the key step elsewhere?"*
   Do not proceed past a proof sketch slide without user confirmation.

---

## SPECIAL RULES FOR THE DISCUSSION CALLBACK SLIDE

The discussion section must include a slide that reuses the exact visual from the
Motivation slide. Flag it clearly in the mockup:

```
FIGURE: [Same as Slide N — Motivation visual]
% CALLBACK: Duarte narrative arc closure — "what is" → "what could be"
```

The verbal narrative for this slide must explicitly close the loop opened in the
motivation: *"Recall the problem we opened with — here is what our results now establish."*

Source: Nancy Duarte — *Resonate* (Wiley, 2010), Sparkline theory.

---

## SPECIAL RULES FOR THE VISUAL ROADMAP SLIDE

The roadmap slide (slide 3) must show the thesis chapter structure as a visual timeline.
Describe the layout:
- Horizontal or vertical progression
- Each chapter as a labeled node
- The current section highlighted (using `\alert{}` or color) as the talk progresses

This slide will be reused with different highlighting at each major section transition
to signal to the audience where they are in the arc.

Source: Nancy Duarte — *Resonate* (Wiley, 2010), Sparkline theory.
https://www.duarte.com/books

---

## MOCKUP COMPLETION

After presenting all slide mockups:

1. Ask the user: *"Does this mockup accurately reflect your thesis and the presentation
   you want to give? Are there slides where the visual description, proof sketch, or
   speaker narrative needs adjustment?"*

2. Address any requested changes before proceeding.

3. Write the approved mockup to `defense_mockup.md` in the thesis folder.

4. Update CLAUDE.md: mark Phase 4 status as `approved`, set mockup file path.

5. Confirm: *"Mockup approved. Writing to defense_mockup.md.
   Invoking Phase 5 — LaTeX Generation."*
