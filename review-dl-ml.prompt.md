# Deep Learning / Machine Learning Review Framework

Domain-specific review checklist for DL/ML codebases. Loaded by the Code Review agent when the project involves training loops, data pipelines, or classical ML. Load alongside `review-methodology.prompt.md` and `evaluation-methodology.prompt.md`.

## 1. Training Loop & Architecture

- Inspect training loop correctness (forward pass, loss calculation, `zero_grad`, backward pass, optimizer step — in that order).
- Check for efficient gradient accumulation and mixed-precision (FP16/BF16) correctness.
- Look for GPU memory leaks (e.g., storing history across batches without calling `.detach()`).
- Verify loss function matches the task (e.g., CrossEntropy for classification, not MSE).

## 2. ML Correctness & Reproducibility

- Ensure random seeds are fixed across all libraries (NumPy, PyTorch/TF, CUDA).
- Verify train/val/test split separation. Look for data leakage (e.g., applying dataset-wide scaling before splitting, or train/val preprocessing mismatch).
- Check class imbalance handling and metric selection validity.
- Verify that evaluation metrics match the stated objective.

### Determinism (seeds alone are not enough)
Fixing seeds does not make a CUDA training run reproducible. Also require, where reproducibility
is claimed: `torch.use_deterministic_algorithms(True)`, `CUBLAS_WORKSPACE_CONFIG=:4096:8`,
`cudnn.benchmark=False`, a seeded DataLoader `worker_init_fn` and `generator`, and pinned library
plus CUDA versions. A reproducibility claim without these is **High** — label it "seeded, not
deterministic".

### Benchmark contamination
Train/val/test separation guards leakage *you* introduced. Where a pretrained or foundation model
is involved, also check that the evaluation set did not leak through **pretraining** — public
benchmarks, scraped corpora, or dataset cards overlapping the eval set. An uncontaminated claim
with no contamination check is **High**.

### Statistical reporting
Apply `review-methodology.prompt.md` §1: state n, report stratified bootstrap CIs and IQM rather
than bare point estimates, report effect size. Averaging runs does not establish significance.
Results must be reported per slice as well as in aggregate.

## 3. Data Loading & Bottlenecks

- Inspect dataloader configurations (e.g., `num_workers`, `pin_memory`, `prefetch_factor`).
- Check for slow loop-based tensor modifications instead of vectorized operations.
- Look for unnecessary `.cpu()` / `.numpy()` round-trips inside training loops.

## 4. Performance Profiling (when review mode is "Performance")

Code inspection alone cannot diagnose performance bottlenecks — you need runtime data. When the user requests a performance review:

1. **GPU utilization & memory** — Use the Bash tool to run `nvidia-smi --query-gpu=utilization.gpu,utilization.memory,memory.used,memory.total --format=csv` to check whether the GPU is actually saturated or idle (indicating a CPU-bound data pipeline).
2. **PyTorch Profiler** — If PyTorch is used, suggest or insert a `torch.profiler.profile()` context manager around the training step and run a short profiling pass (2-5 batches). Inspect the output for:
   - CUDA kernel launch overhead vs. compute time
   - Host-to-device transfer bottlenecks
   - Operators consuming disproportionate time
3. **Throughput baseline** — Measure samples/second or tokens/second by timing a fixed number of batches. Compare against known baselines for the architecture (e.g., a ResNet-50 on a single A100 should sustain ~1200 img/s at FP16).
4. **Memory peak tracking** — Use `torch.cuda.max_memory_allocated()` before and after a training step to identify peak memory. Flag if peak exceeds 80% of GPU VRAM (leaves no headroom for batch size scaling).
5. **CPU profiling** — If GPU utilization is low (<50%), the bottleneck is likely CPU-side. Run `py-spy top --pid <PID>` or suggest `cProfile` to identify slow Python code in data preprocessing.
