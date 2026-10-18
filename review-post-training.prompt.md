# Post-Training Review Framework (SFT, DPO, RLHF, GRPO)

Domain checklist for fine-tuning and preference-optimisation code. Loaded by the Reviewer — and by
the Coder at Step 3 when the blueprint includes fine-tuning — alongside `review-dl-ml.prompt.md`
and `evaluation-methodology.prompt.md`. Classic environment RL stays in `review-rl.prompt.md`.

## Checklist

- **Loss mask matches intent.** TRL's `SFTTrainer` computes loss on completion tokens only by
  default for prompt–completion data; `assistant_only_loss=True` only works if the chat template
  contains `{% generation %}` markers. A mask that trains on prompt or system tokens is **High**. [S1]
- **One chat template** for training and inference, pinned with the tokenizer. A mismatch silently
  degrades the model at serving time. [S1]
- **Preference optimisation records** the reference model, β, and the data's chosen/rejected
  provenance. [S2]
- **Proxy reward is not the goal.** Track a held-out gold metric beside the reward or preference
  score, and stop or roll back when they diverge — over-optimising a proxy reward lowers true
  quality. [S3]
- **Diagnose before tuning.** Apply `evaluation-methodology` §3a (bias, variance, data mismatch)
  before changing hyperparameters, and confirm with `llm-context-grounding` §6 that fine-tuning is
  the right lever at all.

## Sources

| ID | Source | Where the argument is |
|---|---|---|
| S1 | Hugging Face TRL, SFTTrainer — https://huggingface.co/docs/trl/sft_trainer | Loss computation; `assistant_only_loss`; chat templates |
| S2 | Hugging Face TRL, DPOTrainer — https://huggingface.co/docs/trl/dpo_trainer | Reference model; `beta` |
| S3 | Gao et al., Scaling Laws for Reward Model Overoptimization — https://arxiv.org/abs/2210.10760 | Abstract |
