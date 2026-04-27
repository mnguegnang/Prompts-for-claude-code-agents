# Theory Lecture Structure

Template for academic/pedagogical lectures explaining theoretical concepts, research findings, and methodological evolution. Follows the Source-First Principle.

**Applies LaTeX conventions from:** `~/.claude/prompts/latex-conventions.prompt.md`

## File Naming

`[Topic_Name]_Lecture.tex`

## Lecture Syllabus

### A. Introduction & Running Example

Ground the lecture in a concrete dataset or context from the user's problem. Introduce the running example that will be threaded through all subsequent sections.

### B. Theoretical Foundations (Source-First Principle)

Two-phase pedagogy for every major concept:

1. **STEP 1 — The Source:** Present the formal mathematics exactly as defined in the authoritative source papers. Annotate each equation with `\sourcenote{}` citing the originating paper, section, and page.
2. **STEP 2 — The Intuition:** Provide a plain-English translation of the mathematics, applied to the Running Example. Use analogies and diagrams where they aid understanding.

### C. Evolution of Methodologies

Trace the progression from foundational methods to SOTA. Emphasize *why* each advancement was made — what limitation of the prior approach it addressed. Use a chronological or thematic structure depending on the field.

### D. Algorithmic Pseudocode (CONDITIONAL)

Include formal pseudocode using `algorithm2e` **only if** source papers explicitly feature algorithms. Do not invent pseudocode for methods that were not algorithmically specified in the literature.

### E. Limitations & Failure Modes

Identify where methods break down, under what conditions, and cite the papers that discovered or characterized these failure modes. Include known mitigations if documented.

## Post-Generation

After compiling the PDF:
1. Provide a brief critical takeaway summary in chat.
2. Ask: *"Please review the lecture PDF. Are there any concepts or citations you would like me to clarify?"*
3. Await confirmation before proceeding to the next phase.
