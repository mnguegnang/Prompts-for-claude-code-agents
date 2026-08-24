# PhD Defense Agent — Phase 1: Document Reading & Inventory
# File: ~/.claude/prompts/phd-defense-p1-reading.md
# Invoked after Phase 0 is approved. Read phd-defense-context.md first.

---

## PURPOSE

Thoroughly read all documents in the thesis folder and produce a structured inventory
that all subsequent phases will depend on. This phase never generates slides or outlines —
it only extracts and organizes knowledge from the source material.

---

## ACTIONS

### Step 1 — Read All Documents
Use the `Read`, `Glob`, and `Grep` tools to read every file in the thesis folder path
recorded in CLAUDE.md. This includes:
- All thesis chapter files (abstract, introduction, related work, methods, results, conclusion)
- Any past presentation files (.tex, .pdf, .pptx)
- Any figure files — inventory their filenames and formats

### Step 2 — Extract the Full Thesis Inventory
Organize your findings into the following structured sections:

#### A. Thesis Identity
- Full thesis title
- Author name
- Field and subfield
- Central mathematical problem: what is the open question being addressed?
- Why does it matter mathematically and/or for applications?

#### B. Contributions — Stated as Precise Mathematical Claims
List every main contribution exactly as the thesis states it. For Applied Mathematics,
each contribution must be framed as one of:
- A theorem proven (state it)
- An algorithm developed (state its properties: complexity, convergence, correctness)
- A convergence or existence result established
- A new framework, model, or formulation introduced

Source: NPS Defense Outline (Rowe, N.) — "State contributions as claims. Most of the
remainder of your presentation will be the validation of your claims."
https://faculty.nps.edu/ncrowe/phd_defense_outline.htm

#### C. Mathematical Content Inventory
For each named mathematical object, record:

| Label | Type | Statement (brief) | Proof available in folder? |
|---|---|---|---|
| e.g. Thm 3.1 | Theorem | [one-line statement] | Yes / No / Sketch only |

Types: Theorem, Lemma, Definition, Corollary, Proposition, Algorithm, Assumption.

#### D. Related Work — Two NPS Categories
Organize prior work into exactly two categories (do not merge them):
1. **Prior methods addressing the same problem with different approaches** — name researchers,
   summarize what they did, explain specifically why their work did not fully solve the problem.
2. **Prior methods using similar techniques on different problems** — these can be brief if the
   techniques are well-known; note which techniques overlap with your approach.

Source: NPS Defense Outline. https://faculty.nps.edu/ncrowe/phd_defense_outline.htm

#### E. Limitations and Boundary Conditions
List all limitations explicitly stated or implied in the thesis:
- Known boundary conditions under which results fail or degrade
- Assumptions that restrict the generality of the results
- Open problems the work raises but does not resolve

These must be specific, not vague. "Future work is needed" is not acceptable as a limitation.

#### F. Future Work Directions
List specific (not vague) next steps identified in the thesis. For each, note whether
it is a natural extension of the main results or a separate open problem.

#### G. Figure Inventory
Produce a complete table of all figure files found in the folder:

| Filename | Format | Inferred content | Section it belongs to | Vector? |
|---|---|---|---|---|

Flag any figure that is raster format (.png, .jpg) — these cannot be used directly in
the final LaTeX without a note to the user to provide vector versions (.pdf, .eps).

Source: NPS Defense Outline — "Do not use font smaller than 28 point" and figure quality
standards for public defense rooms. https://faculty.nps.edu/ncrowe/phd_defense_outline.htm

#### H. Past Presentation Materials
If any past slides are in the folder, extract:
- What structure was used previously
- Which figures were already prepared for presentation use
- Any speaker notes or timing annotations

---

### Step 3 — Weak Material Detection
Apply the Weak Material Protocol from phd-defense-context.md to every section:
- If the conclusion chapter is less than one page, flag it.
- If no proof or proof sketch exists for a main theorem, flag it.
- If the limitations section is absent or contains only one sentence, flag it.
- If future work is described only in vague terms, flag it.

Report all flags to the user before proceeding.

---

### Step 4 — Write Inventory to CLAUDE.md
Append the full inventory under a `## Document Inventory` section in CLAUDE.md.

---

### Step 5 — Output to User
Present a concise encouraging summary (1 paragraph) confirming your understanding:
- What mathematical problem does the thesis solve?
- What are the main contributions?
- What makes the work non-trivial — what did prior work fail to do?

Then ask: *"Does this accurately represent your thesis? Are there important results or
context I may have missed?"*

Wait for user confirmation. Mark Phase 1 as `approved` in CLAUDE.md only after the user
explicitly confirms the summary is accurate.
