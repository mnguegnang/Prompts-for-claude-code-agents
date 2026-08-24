# PhD Defense Agent — Shared Context & Constants
# File: ~/.claude/prompts/phd-defense-context.md
# Read this file at the start of every phase before acting.

---

## PURPOSE

This file is the single source of truth for all session state, approved decisions, and structural
constants shared across all phases of the PhD Defense Beamer Architect agent. Every prompt file
reads this file first and writes updates back to CLAUDE.md after each phase completes.

---

## CLAUDE.md SCHEMA

Every project using this agent must maintain a `CLAUDE.md` file at the project root with the
following structure. The agent reads this at session start and updates it after every approved
phase transition.

```markdown
# PhD Defense Agent — Session State

## Thesis Metadata
- Title:
- Author:
- Thesis folder path:
- LMU Beamer template path:
- Key non-expert audience member:

## Approved Decisions

### Phase 0 — Initialization
- Status: [ pending | approved ]

### Phase 1 — Document Reading
- Status: [ pending | approved ]
- Thesis summary confirmed by user: [ yes | no ]

### Phase 2 — Crux & One-Sentence Message
- Status: [ pending | approved ]
- Crux statement: |
    [plain-English paragraph — no LaTeX]
- One-sentence message: |
    "In this thesis, I show that __________, which matters because __________."

### Phase 3 — Narrative Strategy & Slide Structure
- Status: [ pending | approved ]
- Narrative strategy selected: [ A | B | C ]
- Per-theorem proof sketch placement:
    - Theorem 1 [label]: [ immediately after | grouped after results | in methods section ]
    - Theorem 2 [label]: [ immediately after | grouped after results | in methods section ]
    - [add entries as needed]
- Per-theorem slide budget:
    - Theorem 1 [label]: [ full treatment | statement + sketch | statement only ] — N slides
    - Theorem 2 [label]: [ full treatment | statement + sketch | statement only ] — N slides
- Figure placement map:
    - Slide N: [filename or PLACEHOLDER — description]
- Slide outline: [written to defense_outline.md]

### Phase 4 — Content Mockup
- Status: [ pending | approved ]
- Mockup file: defense_mockup.md
- Crux correctly captured in proof sketches: [ yes | no ]

### Phase 5 — LaTeX Generation & Validation
- Status: [ pending | approved ]
- Output file: defense_presentation.tex
- Validation result: [ pass | warnings | failures ]
- PDF compiled successfully: [ yes | no ]

### Phase 6 — Q&A Preparation (OPTIONAL)
- Status: [ not started | in progress | complete ]
```

---

## APPROVED DECISIONS REGISTRY

When reading this file, always check the CLAUDE.md in the project root for the current state
of all approved decisions. Never re-ask for information already recorded as approved.

If CLAUDE.md does not exist yet, proceed to Phase 0 initialization.

---

## FILE NAMING CONVENTIONS

| File | Purpose |
|---|---|
| `CLAUDE.md` | Live session state and approved decisions |
| `defense_outline.md` | Approved slide-by-slide outline |
| `defense_mockup.md` | Approved text-based content mockup |
| `defense_presentation.tex` | Final LaTeX Beamer output |

All files are written to the thesis folder path recorded in CLAUDE.md.

---

## PHASE TRANSITION RULE

**Never move to the next phase until the user has explicitly approved the current phase output.**
After each approval, update the corresponding status field in CLAUDE.md to `approved` before
invoking the next phase prompt.

---

## WEAK MATERIAL PROTOCOL

If any section in the thesis folder contains thin or missing content, surface it immediately
using this format — never generate a weak slide silently:

```
⚠️  WEAK MATERIAL DETECTED — [section name]
The content available for this section is insufficient to generate a strong slide.
Specifically: [what is missing and why it matters for the defense].
Options:
  (a) Can you supply additional material for this section?
  (b) I will generate the slide from available material and mark it: % NEEDS REVIEW
```

---

## SESSION RESUMPTION RULE

At the start of any new session:
1. Read CLAUDE.md from the project root.
2. Identify the last approved phase.
3. Report to the user: "Last completed phase: [N]. Shall we continue from Phase [N+1],
   or is there something you would like to revisit?"
4. Never re-ask for information already recorded as approved in CLAUDE.md.
