# Production Operations (shared — PI STAGE 6, Coder Step 6, Reviewer for serving code)

What happens after deployment: detecting drift, responding to incidents, and cutting cost and
latency without losing quality. Covers Ng's skills-map area *Operating in production* [S0].
`evaluation-methodology.prompt.md` §11–12 says what to log and when to roll back; this file says
how to detect change and what to do about it. Each rule cites its source at the end.

## 1. Drift and skew detection

- **One feature code path.** Training and serving compute features through shared code; a
  separate serving reimplementation is a skew bug waiting to happen. [S1 Rule #32]
- **Data expectations are a schema** (types, ranges, missing fraction). Training and serving data
  are both checked against it, and a violation alerts. [S2 Data 1, Monitor 2]
- **Compare logged serving inputs with training inputs**: per-feature min, max, mean, missing
  fraction, and skew counts. For high-dimensional inputs, two-sample tests on a reduced
  representation detect shift best. [S1 §Training-serving skew; S3]
- **Live quality against launch-time bounds.** Alert on sudden drops *and* slow-leak declines.
  Alert when the **lower confidence bound** of a production metric crosses its threshold, and tune
  thresholds so false alarms stay rare enough that alerts are acted on. [S2 Monitor 7; S4 §CI/CD]
- **Track upstream changes**: provider model versions and deprecations, index age, embedding model
  version. Any change re-runs the regression suite (`evaluation-methodology` §9).
- **Training runs** alert on the first NaN or Inf. [S2 Monitor 5]

## 2. Incident response

Before serving, the blueprint names the **disengage mechanism** (feature flag, fallback model,
degraded mode) and its owner. [S5 MANAGE 2.4]

For every incident, including prompt-injection and data-exfiltration events:
1. **Contain** — disengage or degrade; do not debug a live harm.
2. **Triage** from replayed traces (`evaluation-methodology` §11 requires them).
3. **Add the case to the regression suite** so it can never silently recur.
4. **Write it up** in `Project_Notes.md`: trigger, impact, root cause, the regression case added.
[S5 MANAGE 4.1, 4.3]

## 3. Cost and latency optimisation — in this order

Confirm with error analysis that the model, not the pipeline, is the problem before changing it.
Then cut cost and latency in this order, cheapest change first:

1. **Simplify the workflow** — fewer calls, fewer agent steps, code instead of an LLM where the
   output is computable. [S6 §When (and when not) to use agents]
2. **Adapt the prompt** — shorter context, caching (`llm-context-grounding` §5).
3. **Route or cascade** to cheaper models. [S7]
4. **Distil or fine-tune** — only after evals exist to gate it. [S8]

Each step is re-gated with a **paired non-inferiority test** against the frozen STAGE 2.5
thresholds (`evaluation-methodology` §10). A cheaper system that fails a guardrail is a
regression, not a saving. For latency specifically, apply the provider's latency principles
(fewer tokens out, fewer requests, parallelise independent calls). [S9 §Seven principles]

## Sources

| ID | Source | Where the argument is |
|---|---|---|
| S0 | Ng, AI Engineering Skills Map: Building and deploying AI applications — https://www.linkedin.com/pulse/ai-engineering-skills-map-building-deploying-applications-andrew-ng-gyn5e/ | ¶ "Operating in production" |
| S1 | Google, Rules of Machine Learning — https://developers.google.com/machine-learning/guides/rules-of-ml | Rules #29, #32, #37; §Training-serving skew |
| S2 | Breck et al., The ML Test Score — https://research.google.com/pubs/archive/aad9f93b86b7addfea4c419b9100c6cdd26cacea.pdf | Data 1; Monitor 1, 2, 4, 5, 7 |
| S3 | Rabanser et al., Failing Loudly (NeurIPS 2019) — https://arxiv.org/abs/1810.11953 | Abstract |
| S4 | Husain, Evals FAQ — https://hamel.dev/blog/posts/evals-faq/ | §How are evaluations used differently in CI/CD vs. monitoring production? |
| S5 | NIST AI 100-1, AI RMF 1.0 — https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf | MANAGE 2.3, 2.4, 4.1, 4.3 |
| S6 | Anthropic, Building effective agents — https://www.anthropic.com/engineering/building-effective-agents | §When (and when not) to use agents |
| S7 | Chen et al., FrugalGPT — https://arxiv.org/abs/2305.05176 | Abstract |
| S8 | OpenAI, Supervised fine-tuning guide — https://platform.openai.com/docs/guides/distillation | "Only invest in fine-tuning after setting up evals" |
| S9 | OpenAI, Latency optimization — https://developers.openai.com/api/docs/guides/latency-optimization | §Seven principles |
