# GPU Profiling & Parallelism Strategy

Apply these practices during implementation when the blueprint involves GPU training, fine-tuning, or inference at scale.

## GPU Profiling

Before optimizing, **measure**. Insert profiling at the training/inference entry point:

1. **GPU utilization & memory baseline** — Run using the Bash tool:
   ```bash
   nvidia-smi --query-gpu=utilization.gpu,utilization.memory,memory.used,memory.total --format=csv
   ```
2. **PyTorch Profiler** — Wrap a short profiling pass (2–5 batches) with `torch.profiler.profile()`. Inspect:
   - CUDA kernel time vs. CPU overhead
   - Host-to-device transfer bottlenecks
   - Top operators by GPU time
3. **Memory peak tracking** — Insert `torch.cuda.max_memory_allocated()` before and after a training step. Report peak usage as a percentage of total VRAM.
4. **Throughput measurement** — Time a fixed number of batches, compute samples/sec (or tokens/sec for LLMs). Report as baseline before any optimization.
5. **CPU-side bottleneck check** — If GPU utilization is <50%, the bottleneck is CPU-side (data loading, preprocessing). Check `num_workers`, `pin_memory`, and use `py-spy top` if needed.

Report all profiling results to the user with a brief analysis before proposing optimizations.

## Parallelism Strategy Selection

When the blueprint requires distributed training or the model exceeds single-GPU memory, select and justify a parallelism strategy:

- **Data Parallelism (DP / DDP)** — Use when the model fits in a single GPU's memory but training is slow. Each GPU processes a different mini-batch; gradients are synchronized. Prefer `DistributedDataParallel` over `DataParallel` (it avoids the GIL bottleneck and uses NCCL). **Choose this when:** model fits in VRAM, dataset is large, you want near-linear speedup with GPU count.
- **Tensor Parallelism (TP)** — Use when a single layer's parameters exceed one GPU's memory (e.g., large attention matrices in 70B+ models). Splits individual tensors across GPUs within a single node. **Choose this when:** model does NOT fit in a single GPU, low inter-GPU latency is available (NVLink), and the model has large matrix multiplications.
- **Pipeline Parallelism (PP)** — Use when the model has many sequential layers and you want to split by layer across GPUs. **Choose this when:** model depth is the bottleneck, and you can tolerate micro-batch scheduling complexity.
- **FSDP (Fully Sharded Data Parallelism)** — Use when model parameters, gradients, and optimizer states together exceed single-GPU memory, but individual layers fit. Shards everything across GPUs. **Choose this when:** model is 1B–30B parameters, you want DDP-like simplicity with memory savings.

## Decision Protocol

1. Measure single-GPU memory usage via profiling (above).
2. Compare model memory footprint vs. available VRAM.
3. Select the simplest strategy that fits. Present to the user:
   - *"Model requires X GB; GPU has Y GB. Recommending [strategy] because [reason]. Alternative: [other strategy] if [condition]. Approve?"*
4. Log the decision in `Project_Notes.md`.
