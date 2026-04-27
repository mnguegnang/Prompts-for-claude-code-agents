# ML Project Documentation Templates

Reusable documentation standards for ML/GenAI implementation projects.

## Four-Tier Documentation Architecture

Maintain these files strictly separate:

| File | Purpose |
|------|---------|
| `README.md` | Operational setup and execution commands |
| `Developer_Log.md` | Chronological tracker: errors, root causes, fixes |
| `Project_Notes.md` | ADR: deviations from PI blueprint, architectural assumptions, known debt, scaling limits |
| `[Topic]_Code_Lecture.tex` | (Optional, end-of-project) Formal academic lecture on the codebase architecture |

## Developer_Log.md Entry Format

Every time an error is resolved, append:

```markdown
## [YYYY-MM-DD HH:MM] — <Short title>
- **Error:** <exact error message>
- **Root cause:** <diagnosis>
- **Fix:** <what was changed>
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
│   └── test_smoke.py
├── config.yaml
├── .env.example
├── requirements.txt
├── Dockerfile             # If deployment stage specified
├── README.md
├── Developer_Log.md
└── Project_Notes.md
```
