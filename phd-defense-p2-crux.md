# PhD Defense Agent — Phase 2: Crux Extraction & One-Sentence Message
# File: ~/.claude/prompts/phd-defense-p2-crux.md
# Invoked after Phase 1 is approved. Read phd-defense-context.md first.

---

## PURPOSE

This phase runs the two highest-stakes synthesis tasks before any planning or slide design
begins. If either output is wrong, every downstream phase inherits that error. Take the
time to get both right.

The Crux Statement is the intellectual lens through which all proof sketch slides will be
designed. The One-Sentence Message is the structural spine of the entire presentation —
it appears verbatim in three places and must never change after approval.

---

## AUTHORITATIVE FOUNDATIONS

**Paul Halmos — "How to Talk Mathematics" (1974):**
"A public lecture is not a textbook. The purpose of a public lecture is to give the
audience a flavor of the subject. The best way to do this is to present the idea of
the proof, not the proof itself."
The Crux Statement is the operationalization of Halmos's principle: before designing
any proof sketch slide, the agent must identify and confirm with the user what the
*idea* of the proof is — not the mechanics.

**Nature — "How to Make an Outstanding Scientific Presentation" (2023):**
https://www.nature.com/articles/d41586-023-03447-9
Recommends a single bold thesis sentence as the structural anchor of the presentation.

**Stanford DDRIG — Prof. Margot Gerritsen:**
Specifies the exact sentence template: "In this thesis, I show that __________, which
matters because __________."

**Matt Might — "Academic Martial Arts: Defending Your PhD":**
https://matt.might.net/articles/phd-defense-tips/
"The candidate must impart a sense to the committee that she can convey the 'intuition'
behind complex ideas." The Crux Statement is precisely this intuition — stated in
plain English before it is translated into mathematics.

---

## TASK A — CRUX STATEMENT

### What it is
A plain-English paragraph (no LaTeX, no mathematical symbols) that answers:
1. What is the single most important mathematical insight of this thesis?
2. What makes it non-trivial — what is the obstacle that makes this hard?
3. What existing approach does it supersede, extend, or circumvent — and how?

### What it is NOT
- Not a list of contributions (that is the contributions slide)
- Not the theorem statement (that is the theorem slide)
- Not a summary of the thesis (that is the abstract)

It is the *idea* — the intellectual move that makes the work work.

### How to extract it
From the document inventory produced in Phase 1:
1. Identify the theorem or result that is the deepest contribution.
2. Ask: what would a mathematician in a neighboring subfield need to understand to
   appreciate why this result is surprising or difficult?
3. Ask: what is the one step in the proof that is genuinely novel — the "trick" or
   "key idea" that a standard approach would miss?
4. Write the answer to question 3 in plain English as the Crux Statement.

Source: John E. McCarthy — "How to Give a Good Colloquium" (AMS, 1999):
"You should only show the 'crux' of the argument or the main 'trick.' Omit technical
lemmas unless they are the central contribution."

### Output format
```
CRUX STATEMENT:
[Plain-English paragraph, 3–6 sentences, no LaTeX]

Central insight: [one sentence]
Why it is non-trivial: [one sentence]
What it supersedes or extends: [one sentence]
```

Present this to the user and ask:
*"Does this Crux Statement capture the key insight of your thesis accurately? Is the
central trick correctly identified? This will be the foundation for all proof sketch
slides."*

Do not proceed until the user explicitly confirms the Crux Statement is accurate.

---

## TASK B — ONE-SENTENCE MESSAGE

### Template (mandatory — do not deviate)
> "In this thesis, I show that __________, which matters because __________."

### Rules
- The first blank must state the main mathematical result in terms a broad academic
  audience can follow — not a committee-only technical formulation.
- The second blank must state the significance: mathematical, computational, or applied.
  It must answer "so what?" for the non-expert audience member identified in Phase 0.
- The sentence must be comprehensible to the key non-expert recorded in CLAUDE.md.
- This exact sentence will appear verbatim in THREE places in the final presentation:
  1. Slide 2 — the One-Sentence Message slide (immediately after the title slide)
  2. The abstract reference in the introduction slide or motivation context
  3. The conclusion slide — echoed word for word

This verbatim repetition creates the structural spine of the presentation (Stanford
Gerritsen; Duarte, N. — *Resonate*, 2010, Sparkline theory).

### Draft process
1. Take the central contribution from Phase 1 inventory.
2. Translate into plain English using the template.
3. Check: can the non-expert audience member identified in Phase 0 follow this sentence?
   If not, simplify the first blank without losing mathematical precision in meaning.
4. Check: does the second blank answer "so what?" concretely? Avoid "this is important
   for the field" — state specifically what becomes possible, computable, or provable.

### Output format
```
ONE-SENTENCE MESSAGE:
"In this thesis, I show that __________, which matters because __________."

Accessibility check: [Is this sentence followable by the key non-expert? Yes/needs revision]
Significance check: [Does the second blank answer 'so what?' concretely? Yes/needs revision]
```

Present to the user and ask:
*"Do you approve this one-sentence message? Once approved, this sentence will appear
verbatim on slide 2 and in the conclusion. Any refinement must happen now."*

---

## PHASE 2 COMPLETION

Only after the user explicitly approves BOTH the Crux Statement and the One-Sentence
Message:
1. Write both to CLAUDE.md under Phase 2 approved decisions.
2. Mark Phase 2 status as `approved`.
3. Confirm: *"Crux and message locked. Invoking Phase 3 — Narrative Strategy."*
