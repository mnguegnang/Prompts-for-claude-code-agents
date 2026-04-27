# Testing & Quality Gates (Coder Steps 4–5)

Detailed testing protocol and quality gate enforcement for the Coder Agent.

## Dry Run & Cost Estimation (Cost Protection)

Before any script that processes bulk data or makes looping calls to paid APIs:

1. Process exactly **ONE** item, display the result.
2. **Cost Estimation:** After the dry run, calculate and present projected cost before bulk execution:
   - Measure tokens consumed (or API units) for the single item.
   - Multiply: `unit_cost × total_items = estimated_total_cost`.
   - Present: *"Dry run successful. Estimated cost for full run: $X.XX (Y items × $Z.ZZ each). Approve full execution?"*
3. Mock LLM API calls in tests to save costs.

## Test Layers

- **Unit Tests:** Utility functions and data-processing modules.
- **Integration Tests:** Pipeline component connections.
- **Smoke Tests:** End-to-end minimal execution validation.
- **API Tests:** FastAPI endpoint validation (if applicable).
- **GenAI-Specific Tests:** Prompt/template rendering, retrieval output shapes, tool routing logic, Pydantic output schema conformance.

Run all tests: `pytest tests/ -v`

## Quality Gate Sequence

You are **strictly forbidden** from committing code unless it passes all quality gates.

1. Run the full gate sequence using the Bash tool:
   ```bash
   black src/ tests/
   ruff check src/ tests/
   mypy src/
   bandit -r src/ -q
   pytest tests/ -v
   ```
2. If any gate fails: fix the issue, log it in `Developer_Log.md`, and re-run.

## Developer Log Summary

Before presenting the commit, output a brief tally from `Developer_Log.md`:
- *"Developer Log: X errors encountered and resolved. Y open items remaining."*
- This gives the user full context for approving the commit.

## Git Review Loop

Once all gates pass:
1. Run `git status` and `git diff --stat` using the Bash tool.
2. Present a proposed commit message to the user.
3. **Only commit upon explicit user approval.**
