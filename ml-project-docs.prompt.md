# ML Project Documentation Templates

Reusable documentation standards for ML/GenAI implementation projects.

## Documentation Architecture & File Ownership

Maintain these files strictly separate. **Each file has exactly one writing agent** — two agents
appending to the same ground-truth file produces a log that describes code no longer in the tree.

| File | Purpose | Written by | Read by |
|------|---------|-----------|---------|
| `README.md` | Operational setup and execution commands | Coder | everyone |
| `Developer_Log.md` | Chronological tracker: errors, root causes, fixes | **Coder only** | Coder, Reviewer |
| `Review_Log.md` | Review findings and fixes applied by the reviewer | **Reviewer only** | Coder, Reviewer |
| `Project_Notes.md` | ADR: deviations from blueprint, assumptions, known debt, scaling limits | Coder writes; Reviewer appends **only** to the Conflicts section | everyone |
| `Evaluation_Report.md` | Eval spec results, error-analysis taxonomy, metrics with dispersion | Coder | everyone |
| `[Topic]_Code_Lecture.tex` | (Optional) Formal academic lecture on the codebase | Coder | user |

**Reconciliation rule.** On resume, the Coder reads `Developer_Log.md`, then `Review_Log.md`, then
`Project_Notes.md`, in that order. Where a review fix contradicts an approved deviation, the
reviewer records it under `Project_Notes.md` § Conflicts Pending Reconciliation and the Coder
resolves it explicitly — with the user — before continuing. Neither agent silently overrules the
other, and neither edits the other's log.

## Developer_Log.md Entry Format

Every time an error is resolved, append:

```markdown
## [YYYY-MM-DD HH:MM] — <Short title>
- **Error:** <exact error message>
- **Root cause:** <diagnosis>
- **Fix:** <what was changed>
- **Files modified:** <list>
```

## Review_Log.md Entry Format (reviewer-owned)

```markdown
## [YYYY-MM-DD HH:MM] — <Finding title>
- **Severity / Confidence:** <Critical|High|Medium|Low> / <Confirmed|Likely|Uncertain>
- **Evidence:** <file:line + the reproduction: failing test, log line, or command output>
- **Impact:** <cost, safety, statistical validity, latency, correctness>
- **Fix applied:** <what changed, or "recommendation only — awaiting approval">
- **Regression guard:** <baseline N pass / M fail → post-fix N' pass / M' fail; broke: <ids>>
- **Files modified:** <list>
```

## Project_Notes.md Deviation Entry Format

When deviating from the PI Blueprint:

```markdown
## [YYYY-MM-DD] — <Deviation title>
- **Original spec:** <what the blueprint said>
- **Deviation:** <what was done instead>
- **Rationale:** <hardware, library conflict, deprecation, etc.>
- **Approved by user:** yes/no
```

### Project_Notes.md § Conflicts Pending Reconciliation (reviewer-appended)

```markdown
## [YYYY-MM-DD] — <Conflict title>
- **Approved deviation:** <what Project_Notes records>
- **Review finding:** <what the reviewer found, with evidence>
- **Conflict:** <why they cannot both hold>
- **Status:** unresolved — requires user decision
```

## setup.cfg (required by gates 5 and 8)

```ini
[mutmut]
source_paths=src/<your_package>

[tool:pytest]
markers =
    live: makes real API calls; excluded by default, costs money
```

Without `source_paths`, mutmut 3.x refuses to start. Without the marker registration,
`pytest -m "not live"` warns, and errors under `--strict-markers`.

## Project Structure Template

```
project_root/
├── src/
│   ├── __init__.py
│   ├── config.py          # Pydantic V2 settings
│   ├── ...                # Modules per blueprint stages
├── tests/
│   ├── test_unit.py
│   ├── test_integration.py
│   ├── test_smoke.py
│   ├── test_properties.py     # Hypothesis + metamorphic relations
│   ├── test_contract.py       # Recorded provider cassettes
│   ├── cassettes/             # Replayed responses (re-record on SDK bump)
│   └── test_live.py           # @pytest.mark.live — real calls, flag-gated
├── evals/
│   ├── eval_spec.md           # STAGE 2.5 EVALUATION SPEC block
│   ├── datasets/              # production baseline, slices, failure-mode sets
│   ├── run_eval.py
│   └── error_analysis.md      # trace sampling, failure taxonomy, frequency table
├── config.yaml
├── setup.cfg              # [mutmut] source_paths + [tool:pytest] markers (gates 5, 8)
├── .env.example
├── requirements.txt
├── Dockerfile             # If deployment stage specified
├── README.md
├── Developer_Log.md
├── Review_Log.md
├── Evaluation_Report.md
└── Project_Notes.md
```
