# 10 — Unified memory

**Goal:** understand the one thing about this machine that breaks every habit you
brought with you, and learn to measure it correctly.

This is the runbook that explains why you bought the hardware. It is also the one
that will stop you from misreading your own tools for the next six months.

---

## The one-pool model

A conventional GPU workstation has two separate pools of memory:

```
  SYSTEM RAM  ───copy───▶  VRAM
   (64 GB)                (24 GB)
   CPU only               GPU only
```

Weights live on disk, get read into system RAM, then get **copied across PCIe**
into VRAM before the GPU can touch them. VRAM is the hard ceiling. A 70B model at
FP16 does not fit in 24 GB, so you quantize, you offload layers to CPU and accept
the slowdown, or you buy a bigger card.

The GB10 does not work this way. There is **one pool**:

```
   ┌─────────────────────────────┐
   │   UNIFIED MEMORY  128 GB    │
   │      LPDDR5X, one pool      │
   └──────┬───────────────┬──────┘
          │               │
       ◀──┴──▶         ◀──┴──▶
         CPU             GPU
```

CPU and GPU address the same physical memory. There is no copy step, because
there is nowhere to copy *to*. This is why `nvidia-smi` reports
`Memory-Usage: Not Supported` — the field it wants to print does not exist.

Two consequences, in order of how much they will change your work:

**1. Your ceiling is ~120 GB, not 24 GB.** Models that require multi-GPU rigs or
aggressive quantization elsewhere simply load here. This is the entire value
proposition of the machine.

**2. That memory is shared, not additional.** The 121 GiB is CPU *and* GPU
together. A dataloader holding 30 GB of preprocessed batches is 30 GB the model
cannot have. On a discrete-GPU box those two allocations never competed. Here they
do, and nothing warns you.

> **Domain aside — skip this if you work with models daily.**
> Model weights are numbers, stored at some precision. FP16 uses two bytes per
> parameter, so a 70-billion-parameter model needs roughly 140 GB just for weights.
> *Quantization* stores them at lower precision — 8-bit, 4-bit — trading a little
> accuracy for a large drop in memory. Most published model sizes assume some
> quantization; check which one before doing arithmetic.

---

## Measure it correctly

**`nvidia-smi` alone will mislead you here.** Use three commands, and know what
each one is actually counting.

### Whole machine

```bash
free -h
```

This is your GPU memory readout. Watch `available`.

**Status: VERIFIED** — 2026-08-24, before and after loading a 4.7 GB model:

```
               total        used        free      shared  buff/cache   available
Mem:           121Gi       3.9Gi       113Gi        32Mi       5.7Gi       117Gi     <- idle
Mem:           121Gi       9.9Gi       102Gi        98Mi        10Gi       111Gi     <- model resident
```

`used` moved by 6.0 GiB. Nothing about that line says "GPU," and the GPU is
nonetheless where the work is happening.

### Per process

```bash
nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv
```

**Status: VERIFIED** — 2026-08-24, same model resident:

```
pid, process_name, used_gpu_memory [MiB]
152615, /usr/local/bin/ollama, 5303 MiB
```

**This works even though the GPU-level total does not.** It is the command to
reach for when you need to know *which* process is holding memory — the question
`free -h` cannot answer.

### Per model, per runtime

```bash
ollama ps
```

**Status: VERIFIED** — 2026-08-24:

```
NAME             ID              SIZE      PROCESSOR    CONTEXT    UNTIL
llama3:latest    365c0bd3c000    5.8 GB    100% GPU     4096       4 minutes from now
```

`PROCESSOR` is the field worth watching. `100% GPU` means the whole model is on
the GPU. A split like `70%/30% CPU/GPU` means something did not fit and you are
paying for it in tokens per second.

---

## Four numbers, one model, all correct

Here is the trap. The same 4.7 GB model, measured four ways, at the same moment:

| Source | Reads | What it is counting |
|---|---|---|
| `ollama list` | 4.7 GB | compressed weights **on disk** |
| `ollama ps` | 5.8 GB | weights **plus KV cache** at 4096 context |
| `--query-compute-apps` | 5303 MiB (5.18 GiB) | what the **GPU context** holds |
| `free -h` delta | 6.0 GiB | total **system** allocation, including runtime overhead |

**Status: VERIFIED** — all four captured 2026-08-24 in a single session.

None of these is wrong and none of them will ever agree. Two separate reasons:

**Units.** `ollama` reports GB (10⁹). `free`, `du`, and `nvidia-smi` report GiB
and MiB (2³⁰, 2²⁰). The gap is about 7% and it grows with size — at 400 GB it is
nearly 30 GB of apparent discrepancy, which is more than enough to send you
hunting for a leak that does not exist.

**Scope.** Disk footprint, weights in memory, KV cache, and runtime overhead are
four different quantities. Ask which one a number refers to before comparing it to
another number.

> **Domain aside — skip this if you work with models daily.**
> The *KV cache* stores intermediate attention values for tokens already
> processed, so they need not be recomputed for every new token. It grows with
> context length, and at long contexts it can rival the weights themselves for
> size. This is why the same model occupies different amounts of memory depending
> on how you configured it.

---

## Practical sizing

**Budget from `available`, not `total`.** Idle overhead on the reference machine
is roughly 4 GiB (desktop, Xorg, services). VERIFIED 2026-08-24.

Rough working ceiling: **~115 GiB**, less whatever else you are running.

Before loading something large:

```bash
free -h                # what is actually free right now
df -h /                # weights have to land on disk first
ollama ps              # is something already resident?
```

A model resident from an earlier session still holds memory. Ollama releases it on
a timeout — the `UNTIL` column in `ollama ps` shows when. If you need it gone
sooner, stop that specific model rather than restarting the service:

```bash
ollama stop <model-name>
```

### When you overcommit

Swap is the warning sign. From runbook 00, healthy idle is `Swap: 0B` used.

```bash
free -h | grep Swap
```

Swap in active use means the unified pool is oversubscribed and pages are moving
to NVMe. Performance does not degrade gracefully here — it collapses. You will
notice in tokens per second long before you think to check this number.

There are 15 GiB of swap configured. **Do not treat it as 15 GiB of extra
capacity.** It is a safety margin that prevents a hard out-of-memory kill; it is
not usable model memory.

---

## What this changes about your habits

| Habit from discrete GPUs | Here |
|---|---|
| Check VRAM before loading | Check `free -h` — same job, different tool |
| Quantize to fit in VRAM | Often unnecessary. Try FP16/BF16 first |
| Offload layers to CPU for big models | Meaningless. There is one pool |
| CPU RAM and VRAM budgeted separately | One budget. Dataloaders compete with weights |
| `nvidia-smi` shows memory pressure | It does not. `free -h` does |

The one genuinely new discipline: **watch your dataloader and preprocessing
memory.** On a discrete-GPU machine, host memory was effectively free and never
competed with the model. Here it does, and a preprocessing step that quietly
buffers 40 GB is now capable of pushing your model into swap.

---

## When it looks wrong

| Symptom | Check |
|---|---|
| `Memory-Usage: Not Supported` | Expected. Not a fault. |
| Model far slower than expected | `ollama ps` — is `PROCESSOR` 100% GPU? |
| Out of memory well under 120 GB | Something else is resident. `free -h`, `ollama ps` |
| Reported sizes disagree | Units (GB vs GiB) or scope. See the four-numbers table |
| Swap in use | Oversubscribed. Reduce context length or model size |
| Everything crawls, no error | Check swap first, before anything else |

---

## Status

| Claim | Status | Verified against |
|---|---|---|
| `free -h` before/after a model load | **VERIFIED** | 2026-08-24 · driver 580.159.03 · CUDA 13.0 · ollama 0.12.6 |
| `--query-compute-apps` reports per-process GPU memory | **VERIFIED** | same |
| `ollama ps` output and `PROCESSOR` field | **VERIFIED** | same |
| Four-number table, all four figures | **VERIFIED** | same, single session |
| ~4 GiB idle overhead | **VERIFIED** | same, desktop running |
| Swap collapse behaviour under oversubscription | **UNVERIFIED** | Not deliberately reproduced — doing so requires pushing the machine into swap on purpose. General Linux behaviour; confirm before relying on the description. |
| "Often unnecessary to quantize" | **UNVERIFIED** | Follows from the memory ceiling but not benchmarked here. Runbook 40 tests it. |

Outputs are sanitized per the placeholder convention in `CONTRIBUTING.md`.
