# AI Review Methodology, Evaluation & Observability Standards

Cross-cutting review standards applied to ALL AI project reviews regardless of domain. Loaded alongside any domain-specific framework.

## 1. Experiment Comparison Intelligence

If the user asks "which approach is better?", verify the comparison is scientifically valid. Both approaches must share:
- Same data splits, batch size, hardware, seeds, and stopping criteria.
- Same preprocessing pipeline and evaluation cadence.
- Results must be averaged across multiple runs/seeds to prove statistical significance.

Flag any comparison that violates these conditions as **High severity**.

## 2. Evaluation Setup

AI projects fail due to weak evaluation. Check if:
- Metrics match the product goal (e.g., ROC-AUC/F1 for classification, NDCG/MRR for retrieval, tool success rate for agents).
- Offline evaluation aligns with real-world usage.
- Baselines are meaningful and benchmark comparisons are fair.
- Test sets are truly held-out and never seen during training or hyperparameter tuning.

## 3. Eval Artifact Inspection

When evaluation outputs exist in the project, **inspect them directly** — do not rely solely on the code:
- **CSV / JSON lines**: Load with pandas or jq to verify reported metrics match actual data (e.g., `python -c "import pandas as pd; df = pd.read_csv('results.csv'); print(df.describe())"`).
- **W&B / MLflow runs**: Check for run configs, hyperparameter logging completeness, and metric consistency across runs.
- **TensorBoard logs**: Verify loss curves are not diverging, learning rate schedules match code.
- **Training logs**: Inspect for NaN losses, gradient explosion warnings, OOM errors, or suspiciously flat metrics.
- Flag any mismatch between code logic and reported results as **Critical** (possible data leakage or evaluation bug).

## 4. Workflow & Observability

Code is only half the system. Review the workflow for:
- Missing structured logs, latency breakdowns, or token/cost monitoring.
- Lack of per-step tracing in Agent systems.
- Experiment tracking practices (MLflow, Weights & Biases).
- Data versioning, checkpoint strategies, and rollback readiness.

## 5. Software Security Checklist

ML projects are software projects. Apply these checks to every review:

### Secrets & Credentials
- Scan for hardcoded API keys, tokens, passwords, or connection strings in source files, configs, and notebooks.
- Verify `.env` files are in `.gitignore`. Flag any committed `.env` as **Critical**.
- Check that secrets are loaded from environment variables or a secrets manager, never from literals.

### Dependency Security
- Check for pinned dependency versions (`requirements.txt`, `pyproject.toml`, `package.json`). Unpinned deps are **Medium**.
- Flag known-vulnerable packages if version is visibly outdated (e.g., `PyYAML < 6.0`, `requests < 2.31`, `torch < 2.0` with known CVEs).
- Check for `pip install --trusted-host` or `--no-verify` flags that disable TLS verification — flag as **Critical**.

### Input Validation & Serialization
- Flag use of `pickle.load()`, `torch.load()` without `weights_only=True`, or `yaml.load()` without `SafeLoader` as **Critical** (arbitrary code execution risk).
- Check file path handling for path traversal (`../`) when accepting user-provided paths.
- Verify that user-facing APIs validate and sanitize inputs at system boundaries.

### Network & File Access
- Flag `subprocess.run(shell=True)` or `os.system()` with user-controlled inputs as **Critical**.
- Check for `eval()` or `exec()` on untrusted data.
- Verify that HTTP requests use timeouts and validate TLS certificates.

## 6. Severity Definitions

Classify findings using these exact definitions:

| Severity | Definition |
|----------|-----------|
| **Critical** | Security flaw, data leakage, invalid evaluation, dangerous agent behavior |
| **High** | Major performance issue, serious maintainability risk, reproducibility failure |
| **Medium** | Weak abstractions, test gaps, inefficiency, code smell |
| **Low** | Style issues, minor duplication, naming improvements |
