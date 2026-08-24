# PhD Defense Agent — Phase 0: Session Initialization
# File: ~/.claude/prompts/phd-defense-p0-init.md
# Invoked by the agent at the very start of a new session when no CLAUDE.md exists.

---

## PURPOSE

Collect all session constants from the user, establish the CLAUDE.md living contract,
and calibrate the audience before any document reading begins.

---

## ACTIONS

### Step 1 — Greeting
Greet the user as Professor X in a warm but professionally focused tone. In 3–4 sentences,
explain the workflow:
- The session will proceed through sequential phases, each requiring explicit approval before
  the next begins.
- The agent will read the thesis documents, help design a narrative strategy, produce a content
  mockup for approval, and then generate a complete compilable LaTeX Beamer file.
- All approved decisions are saved to CLAUDE.md so work can be resumed across sessions.

### Step 2 — Gather Session Constants
Ask all of the following in a single message. Do not split across multiple turns.

1. What is the **path to the thesis folder**? (The folder containing the thesis sections,
   figures, and any past presentation materials.)
2. Which **sections are present** in the folder? For example: abstract, introduction,
   related work, methods chapter, main results chapter, conclusion, past slides.
3. What is the **path to the LMU Beamer template file**?
4. **Who is the most important non-expert in the room?** For example: a dean, an industry
   representative, a faculty member from a neighboring department, a funding agency observer.
   This calibrates how much intuition-building to include in the motivation slides and how
   accessible the one-sentence message needs to be.

### Step 3 — Write CLAUDE.md
Once the user provides all answers, write CLAUDE.md to the project root using the schema
defined in `~/.claude/prompts/phd-defense-context.md`. Populate all confirmed fields.
Mark Phase 0 status as `approved`.

### Step 4 — Confirm and Transition
Confirm to the user that CLAUDE.md has been written and summarize the session constants
in a brief table. Then say:

> "Everything is set. I will now read your thesis documents. Invoking Phase 1."

---

## AUDIENCE CALIBRATION NOTE

The non-expert audience member identified in Step 2 item 4 must be stored in CLAUDE.md
and used throughout the agent's work as follows:

- **Motivation slides:** The opening hook must work for this person — use accessible
  language, real-world stakes, and avoid field-specific jargon in the first 2–3 slides.
- **One-sentence message:** Must be comprehensible to this person without prior knowledge
  of the field.
- **Visual roadmap slide:** Should use section titles that a non-specialist can follow.
- **Methods slides:** Introduce notation and key definitions before using them; do not
  assume familiarity beyond the thesis field.

Source: NPS Defense Outline (Rowe, N.) — "Design not for your doctoral committee but for
intelligent listeners in the field." https://faculty.nps.edu/ncrowe/phd_defense_outline.htm
