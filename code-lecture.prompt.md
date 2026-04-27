# Code Lecture Structure

Template for post-implementation educational lectures explaining codebase architecture, design patterns, and testing strategy. Bridges the gap between "it works" and "I understand why it works."

**Applies LaTeX conventions from:** `~/.claude/prompts/latex-conventions.prompt.md`

## File Naming

`[Topic_Name]_Code_Lecture.tex`

## Lecture Syllabus

### A. Architecture Overview

- Module map: how `src/` packages are organized and why.
- Data flow: trace a request/input from entry point through processing to output.
- Key abstractions: interfaces, base classes, config objects, and their responsibilities.
- Dependency graph: which modules depend on which, and how coupling is minimized.

### B. Design Patterns & Rationale

Explain each pattern used, *why* it was chosen, and cite authoritative sources:

- **Config-driven design** — Pydantic V2 settings, `.env` + `config.yaml` separation (cite: Twelve-Factor App methodology).
- **Dependency injection** — how components receive their dependencies (cite: GoF, Clean Architecture).
- **Separation of concerns** — module boundaries and single-responsibility adherence (cite: PEP 8, SOLID principles).
- **Additional patterns** as applicable (Repository pattern, Strategy pattern, Factory, etc.).

### C. Key Code Walkthrough

For each critical function or class:
1. **What** it does (one sentence).
2. **Why** it exists (what problem it solves).
3. **How** it works (step-by-step logic, referencing line-level code).
4. **Interactions** — which other modules call it or are called by it.

Use `lstlisting` or `minted` LaTeX environments for inline code excerpts.

### D. Testing Strategy

- How the test layers (unit, integration, smoke, API, GenAI-specific) validate correctness.
- What each layer catches that others cannot.
- Mock strategy: what is mocked, why, and how mock fidelity is maintained.
- Coverage gaps acknowledged (if any).

## Post-Generation

After compiling the PDF:
1. Provide a brief summary of key architectural insights in chat.
2. Ask: *"Would you like me to elaborate on any section or add coverage for additional modules?"*
