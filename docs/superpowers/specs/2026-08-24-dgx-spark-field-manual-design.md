# DGX Spark Field Manual — Design

**Date:** 2026-08-24
**Host:** proteus (DGX Spark, GB10)
**Repo root:** `/home/frank/operations`

## Purpose

Build a working documentation set for operating a DGX Spark that (a) is useful to
Frank immediately and (b) accumulates into a publishable field manual for Spark
owners.

The stated problem is under-utilization: the hardware is capable and heavily
provisioned, but nothing is running. Every Docker container on the box is
`Exited`, the oldest by ten months. The gap is not capability — it is an
operational loop. The documentation is designed to close that gap, and the book
is the byproduct.

## Audience and voice

Owner-operators of a DGX Spark. Terse, prescriptive, command-first. Every page
assumes the reader has the hardware in front of them and wants to do something
with it today. Explanation is included only where it changes a decision.

Troubleshooting is a first-class section, not an appendix. It is the part of a
field manual that earns trust.

## Verification policy

The book's only durable advantage over vendor documentation is that it is true.
Every claim carries an explicit status:

- **VERIFIED** — run on this machine. Records date, host, and relevant versions
  (driver, CUDA, image tag). Quotes real output.
- **UNVERIFIED** — taken from vendor docs or general knowledge, not tested here.
  Cites its source and states why it was not tested (e.g. requires a second
  Spark, requires a multi-hour training run).
- **BROKEN** — attempted and failed. Records the exact error text and the
  workaround, if one exists.

Rules:

1. Never present an invented terminal transcript as real output. If output is
   not quoted from this machine, the block is UNVERIFIED.
2. Record the versions a VERIFIED claim was tested against. Claims age; version
   stamps make the aging visible.
3. UNVERIFIED entries are the writing backlog. `book/OUTLINE.md` tracks them.

## Baseline machine state (verified 2026-08-24)

| Property | Value |
|---|---|
| Hostname | proteus |
| SoC | NVIDIA GB10 |
| CPU | 20 cores — 10x Cortex-X925 + 10x Cortex-A725, aarch64 |
| Memory | 121 GiB unified, 15 GiB swap |
| Storage | 3.7 TB NVMe, 1.6 TB used (45%) |
| OS | Ubuntu 24.04.4 LTS, kernel 6.17.0-1026-nvidia |
| Driver | 580.159.03 |
| CUDA | 13.0 |
| Network | enP7s7 wired, 192.168.1.171/24; wlP9s9 down |

Installed: Docker + nvidia-ctk, Ollama (binary present, store empty), LM Studio,
ComfyUI, JupyterLab, ClearML, VS Code, tmux, nvtop.

Notable images: `nvcr.io/nvidia/vllm:25.09-py3`,
`nvcr.io/nvidia/tensorrt-llm/release:spark-single-gpu-dev` (37 GB),
`nvcr.io/nvidia/pytorch:25.09-py3`, `local/llama.cpp:server-cuda`,
`milvusdb/milvus:v2.5.15-gpu-arm64`, `flux-train`, `flux-comfyui`.

### Storage breakdown (verified 2026-08-24)

| Location | Size |
|---|---|
| `~/comfy` | 408 G |
| Docker local volumes | 249 G |
| Docker images | 101.5 G (46.6 G reclaimable) |
| `~/jupyterlab` | 77 G |
| `~/.lmstudio` | 73 G |
| `~/.cache/huggingface` | 25 G |
| Stopped containers | 19.6 G (100% reclaimable) |
| Docker build cache | 13.5 G (0.9 G reclaimable) |
| `~/.ollama` | 16 K |

Approximately 67 GB is reclaimable (46.6 images + 19.6 containers + 0.9 build cache) with no judgment calls. **Reclamation is
documented, not executed.** No destructive command is run against this machine
as part of authoring. Runbook 30 presents the commands and their consequences;
running them is Frank's decision, separately.

## Structure

```
/home/frank/operations/
  README.md              entry point, status legend, navigation
  MACHINE.md             verified inventory of proteus + drift log
  runbooks/              task-oriented: "I want to ___"
  troubleshooting/
    INDEX.md             symptom -> file, greppable by error string
    <symptom>.md         one file per failure mode
  book/
    OUTLINE.md           chapter map -> source files, gap tracker
  scripts/               verified helpers, referenced by runbooks
  docs/superpowers/specs/  design documents (this file)
```

Three layers, three moments of use:

- **runbooks/** — opened while working. Task-oriented.
- **troubleshooting/** — opened while stuck. Symptom-oriented.
- **book/** — opened while writing. Points at the other two; never copies them.

The separation exists because the moment of use differs. A reader debugging a
crash at 11pm will not read a chapter; they will grep an error string. A reader
setting up a serving stack wants an ordered procedure. Same material, different
retrieval path.

## Runbooks

| File | Covers |
|---|---|
| `00-baseline-health.md` | What healthy looks like; the 60-second check; version pinning and how to know when something drifted |
| `10-unified-memory.md` | GB10 has no discrete VRAM; `nvidia-smi` reports `Memory-Usage: Not Supported`; how to actually measure GPU memory and predict whether a model fits |
| `20-thermals-power.md` | Behavior under sustained load, throttling, interpreting the power and clock numbers, what the small chassis implies for long runs |
| `30-storage-models.md` | The storage table; duplication across LM Studio / HF cache / ComfyUI; reclamation commands and their consequences (documented, not run) |
| `40-serving-llms.md` | Choosing among vLLM, llama.cpp, TensorRT-LLM, LM Studio, Ollama; standing up an OpenAI-compatible endpoint; making it reachable |
| `50-finetuning.md` | LoRA/QLoRA, Unsloth, LLaMA-Factory; sizing a run against 121 GB unified memory; checkpointing and resumption |
| `60-image-video.md` | ComfyUI and FLUX; managing the 408 GB footprint; workflow and model hygiene |
| `70-containers-arm64.md` | NGC images, nvidia-ctk wiring, why `--ipc=host` matters, aarch64 wheel availability, compute capability and prebuilt-wheel mismatch |
| `80-remote-access.md` | Reaching the box from other machines; services that survive reboot |
| `90-keeping-it-running.md` | The operational loop: what runs always, what runs on demand, how to notice when something stopped. The direct answer to "everything is Exited." |

Runbook 90 is the one that addresses the stated problem. The others describe
capability; 90 describes habit.

### Ordering

Runbooks 00, 10, and 30 are written first. They are the foundation: 00 tells you
whether the machine is well, 10 corrects the single most common wrong mental
model for this hardware, and 30 delivers an immediate concrete win. The
remaining runbooks are written in the order Frank's work requires them.

Frank's primary workloads, in his own priority order: local LLM inference host,
fine-tuning and training, image and video generation. Runbooks 40, 50, and 60
map to these. Runbook 70 is a shared dependency of all three and is written when
the first of them needs it.

## Troubleshooting

One file per symptom. Filenames describe what the reader *sees*, not the
underlying cause — the cause is what they are trying to discover.

`INDEX.md` maps literal error strings to files, so that
`grep -ri "core dumped" troubleshooting/` lands the reader somewhere useful.

Entries are seeded from real failures encountered on this machine or documented
with a cited source. No invented failure modes: a troubleshooting section full
of plausible-sounding problems that nobody has hit is worse than a short one.

Each entry carries: symptom (verbatim error text), affected versions, cause,
fix, and — where the fix is not fully understood — an honest statement of that.

## Book view

`book/OUTLINE.md` holds the chapter map. Each chapter names its source runbook
and troubleshooting files. It duplicates no content.

It also tracks the gap list: every UNVERIFIED claim across the corpus, which
constitutes the writing and testing backlog. A chapter is publication-ready when
its gap list is empty or every remaining gap is explicitly justified.

## Scope boundaries

In scope: single-Spark operation of the three named workloads, plus the
supporting concerns (memory, thermals, storage, containers, access, uptime).

Out of scope for now, recorded so the boundary is deliberate:

- Multi-Spark clustering over ConnectX. Requires a second unit; would be
  permanently UNVERIFIED. The NVIDIA playbook is already cloned locally if this
  changes.
- Agent and RAG development stacks (Milvus, workbench). Frank deprioritized this
  relative to the other three workloads.
- Publishing pipeline (typesetting, layout, distribution). The manuscript is
  Markdown; production is a separate project once content exists.

## Success criteria

1. Frank opens a runbook during ordinary work without being prompted to.
2. Every command in the corpus is either VERIFIED against a stated version or
   visibly marked otherwise. No unmarked claims.
3. Runbook 90 exists and describes a loop Frank actually follows, such that the
   "everything is Exited" state does not recur silently.
4. `book/OUTLINE.md` gives an honest, current answer to "how much of this book
   is real?"
