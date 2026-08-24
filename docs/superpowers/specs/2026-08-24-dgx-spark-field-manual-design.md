# DGX Spark Field Manual — Design

**Date:** 2026-08-24 · **Revised:** 2026-08-24 (audience inversion, distribution)
**Reference machine:** DGX Spark (GB10) · specifics recorded in `MACHINE.md`

Strategic decisions and their rationale live in `docs/decisions/`. This document
covers what is being built; those cover why it is licensed and positioned the
way it is.

## Purpose

Build a working documentation set for operating a DGX Spark that (a) is useful
to the author immediately and (b) accumulates into a publishable field manual.

The stated problem is under-utilization: the hardware is capable and heavily
provisioned, but nothing is running. Every Docker container on the box is
`Exited`, the oldest by ten months. The gap is not capability — it is an
operational loop. The documentation is designed to close that gap; the book is
the byproduct.

## Audience and voice

See decisions `0005` (audience inversion) and `0007` (two reader profiles).

Two readers approach this hardware from opposite directions:

| Profile | Knows | Needs explained |
|---|---|---|
| **A — AI practitioner** (primary) | the model | the machine |
| **B — Home labber** (secondary) | the machine | the model |

**Profile A is primary**: a data scientist or AI engineer, often from the
Windows ecosystem, **expert in the model and a novice in the machine**. This
inverts the usual explanation policy:

- **Never explain** LoRA, quantization, attention, KV cache, batch size,
  checkpointing. The reader knows these better than the author.
- **Always explain** systemd, mounts, `$PATH`, permissions, service users,
  `journalctl`, what Docker does to the filesystem, why a command needs `sudo`.

Command-first and terse, but no command appears unexplained: what it does, what
correct output looks like, what to do when the output differs.

**Profile B is served asymmetrically** (decision 0007). Substrate explanations
are inline main text. Domain explanations — quantization, KV cache, model
sizing — are **marked, skippable asides**, never inline:

> **Domain aside — skip this if you work with models daily.**
> Quantization reduces the numeric precision of a model's weights…

An aside must be skippable at a glance, never longer than a short paragraph, and
never load-bearing. If a procedure cannot be followed without it, it is
misclassified and belongs in the main text.

**GUI parity.** Where LM Studio, ComfyUI, JupyterLab, VS Code Remote, or RDP is
the better tool, say so. The author reaches this machine over `xrdp` from a
Windows box — a legitimate pattern the vendor docs never acknowledge. The
terminal is not a hazing ritual.

Troubleshooting is a first-class section, not an appendix. It is the part of a
field manual that earns trust.

## Verification policy

The book's only durable advantage over vendor documentation is that it is true.
Every claim carries an explicit status:

- **VERIFIED** — run on this machine. Records date, host, and relevant versions
  (driver, CUDA, image tag). Quotes real output.
- **UNVERIFIED** — from vendor docs or general knowledge, not tested here. Cites
  its source and states why it was not tested.
- **BROKEN** — attempted and failed. Records the exact error text and the
  workaround, if one exists.

Rules:

1. **Never present an invented terminal transcript as real output.** If output is
   not quoted from this machine, the block is UNVERIFIED and shows no output.
2. Record the versions a VERIFIED claim was tested against. Claims age; version
   stamps make the aging visible.
3. UNVERIFIED entries are the writing backlog. `book/OUTLINE.md` tracks them.

This policy matters *more* for a novice-substrate audience, not less: a reader
who cannot yet distinguish a correct command from a plausible one is entirely
dependent on the manual being right.

## Reference machine (verified 2026-08-24)

Hostnames, LAN addresses, network-interface names, GPU bus IDs, and user paths
are deliberately absent from committed material — see the placeholder convention
in `CONTRIBUTING.md`. Hardware specifications, package versions, and measured
sizes are kept, because they are common to the platform rather than identifying.

| Property | Value |
|---|---|
| SoC | NVIDIA GB10 |
| CPU | 20 cores — 10x Cortex-X925 + 10x Cortex-A725, aarch64 |
| Memory | 121 GiB unified, 15 GiB swap |
| Storage | 3.7 TB NVMe, 1.6 TB used (45%) |
| OS | Ubuntu 24.04.4 LTS, kernel 6.17.0-1026-nvidia |
| Driver | 580.159.03 |
| CUDA | 13.0 |
| Network | wired up (`enP*`), Wi-Fi down (`wlP*`) — interface names vary |
| Remote access | xrdp active and enabled; RDP from a Windows host |

Installed: Docker + nvidia-ctk, Ollama (systemd service, active), LM Studio,
ComfyUI, JupyterLab, ClearML, VS Code, tmux, nvtop.

Enabled services: `docker`, `nv-docker-gpus`, `ollama`, `xrdp`, `xrdp-sesman`.

Notable images: `nvcr.io/nvidia/vllm:25.09-py3`,
`nvcr.io/nvidia/tensorrt-llm/release:spark-single-gpu-dev` (37 GB),
`nvcr.io/nvidia/pytorch:25.09-py3`, `local/llama.cpp:server-cuda`,
`milvusdb/milvus:v2.5.15-gpu-arm64`, `flux-train`, `flux-comfyui`.

### Storage breakdown (verified 2026-08-24)

| Location | Size | Note |
|---|---|---|
| `~/comfy` | 408 G | contents not yet examined |
| `/usr/share/ollama/.ollama` | **396 G** | **invisible from home; see below** |
| Docker local volumes | 249 G | 1.1 G reclaimable |
| Docker images | 101.5 G | 46.6 G reclaimable |
| `~/jupyterlab` | 77 G | |
| `~/.lmstudio` | 73 G | |
| `~/.cache/huggingface` | 25 G | |
| Stopped containers | 19.6 G | 100% reclaimable |
| Docker build cache | 13.5 G | 0.9 G reclaimable |
| `~/.ollama` | 16 K | misleading — not the model store |

Reclaimable without judgment calls: **67 GB** (46.6 images + 19.6 containers +
0.9 build cache).

**The Ollama finding is the manual's founding example.** `~/.ollama` is 16 K and
appears empty, but `ollama list` reports ~423 GB of models and the real store is
`/usr/share/ollama/.ollama` at 396 GiB — a quarter of the drive — because Ollama
runs as a systemd system service under the `ollama` account, whose home is
`/usr/share/ollama`, mode `750`, unreadable by the human user. Three Linux facts,
no AI facts, 396 GiB hidden. (The 423 vs. 396 gap is unit confusion, not
missing data: `du -sh` reports GiB, Ollama reports GB. 396 GiB = 425 GB. This is
itself a substrate detail worth a paragraph in runbook 05.) Full write-up in
decision 0005.

**Reclamation is documented, not executed.** No destructive command is run
against this machine as part of authoring. Runbook 30 presents the commands and
their consequences; running them is a separate decision by the author.

## Structure

```
<repo-root>/
  README.md              entry point, status legend, navigation
  LICENSE                dual-license pointer
  LICENSE-code           Apache-2.0 (canonical)
  LICENSE-prose          CC BY-NC-SA 4.0 (canonical)
  NOTICE                 attribution, incl. NVIDIA playbooks
  CONTRIBUTING.md        contribution guide + CLA
  MACHINE.md             YOUR machine's inventory + drift log (gitignored)
  MACHINE.example.md     template to copy
  runbooks/              task-oriented: "I want to ___"
  troubleshooting/
    INDEX.md             symptom -> file, greppable by error string
    <symptom>.md         one file per failure mode
  book/
    OUTLINE.md           chapter map, gap tracker, perishable/durable split
  scripts/               verified helpers, referenced by runbooks
  docs/
    decisions/           strategic decision records
    superpowers/specs/   design documents (this file)
```

Three content layers, three moments of use:

- **runbooks/** — opened while working. Task-oriented.
- **troubleshooting/** — opened while stuck. Symptom-oriented.
- **book/** — opened while writing. Points at the other two; never copies them.

The separation exists because the moment of use differs. A reader debugging a
crash at 11pm will not read a chapter; they will grep an error string. A reader
setting up a serving stack wants an ordered procedure.

`scripts/` stays empty until a runbook genuinely needs a helper. Scripts are not
invented to fill the directory.

## Runbooks

Every runbook states its scope at the top — **Spark-specific** or **general** —
per decision 0006.

| File | Covers |
|---|---|
| `00-baseline-health.md` | What healthy looks like; the 60-second check; version pinning and how to notice drift |
| `05-linux-for-ai-people.md` | The substrate: systemd and services, mounts, permissions and service users, `$PATH`, `journalctl`, what Docker does to your filesystem. Written so it could stand alone (decision 0006) |
| `10-unified-memory.md` | GB10 has no discrete VRAM; `nvidia-smi` reports `Memory-Usage: Not Supported`; how to actually measure GPU memory and predict whether a model fits |
| `20-thermals-power.md` | Behavior under sustained load, throttling, interpreting power and clock numbers, what the chassis implies for long runs |
| `25-power-noise-placement.md` | **Home lab.** Power draw, heat, fan noise, where to physically put it; always-on economics — idle draw times electricity rate, 24/7 vs. on-demand |
| `30-storage-models.md` | The storage table; **storage you cannot see** (the 396 GB); duplication across LM Studio / HF cache / ComfyUI; reclamation commands and their consequences (documented, not run) |
| `40-serving-llms.md` | Choosing among vLLM, llama.cpp, TensorRT-LLM, LM Studio, Ollama; standing up an OpenAI-compatible endpoint; making it reachable |
| `50-finetuning.md` | LoRA/QLoRA, Unsloth, LLaMA-Factory; sizing a run against 121 GB unified memory; checkpointing and resumption |
| `60-image-video.md` | ComfyUI and FLUX; managing the 408 GB footprint; workflow and model hygiene |
| `70-containers-arm64.md` | NGC images, nvidia-ctk wiring, why `--ipc=host` matters, aarch64 wheel availability, compute capability vs. prebuilt wheels |
| `80-remote-access.md` | Reaching the box from other machines — RDP, SSH, VS Code Remote, web UIs. **Home lab:** LAN integration — reverse proxy, DNS, VPN, mDNS, certificates; sharing the endpoint with family, other machines, other services |
| `90-keeping-it-running.md` | The operational loop: what runs always, what runs on demand, how to notice when something stopped. **Home lab:** backups, UPS behaviour, coexisting with a NAS / Proxmox host / Home Assistant |

**Runbook 90 has its worked example already.** Every hand-started `docker run`
container on this machine is stopped, some for ten months, while `ollama.service`
still runs — because the installer registered it with systemd. On Windows the
reader would have made it a Service and never thought about it again. Same
concept, different name, nobody supplied the name. That is the chapter.

### Ordering

Written first: **00, 05, 10, 30.** These are the foundation — 00 says whether the
machine is well, 05 supplies the substrate vocabulary the rest depends on, 10
corrects the most common wrong mental model for this hardware, and 30 delivers an
immediate concrete win plus the founding example.

Then 40, 50, 60 in the author's priority order (inference host, fine-tuning,
image/video). 70 is a shared dependency, written when the first of them needs it.
80 and 90 close the loop.

25 sits outside that sequence. It is Profile B material (decision 0007) and does
not block anything, so it is written when convenient — though it is cheap, since
measuring idle draw and noise requires only patience and a meter.

## Troubleshooting

One file per symptom. Filenames describe what the reader *sees*, not the
underlying cause — the cause is what they are trying to discover.

`INDEX.md` maps literal error strings to files, so that
`grep -ri "core dumped" troubleshooting/` lands somewhere useful.

Entries come from real failures encountered on this machine or documented with a
cited source. No invented failure modes: a troubleshooting section padded with
plausible-sounding problems nobody has hit is worse than a short one.

Each entry carries symptom (verbatim error text), affected versions, cause, fix,
and — where the fix is not fully understood — an honest statement of that.

## Book view

`book/OUTLINE.md` holds the chapter map. Each chapter names its source runbook
and troubleshooting files and duplicates no content.

It tracks two lists:

1. **The gap list** — every UNVERIFIED claim in the corpus. This is the writing
   and testing backlog. A chapter is publication-ready when its gaps are empty or
   each remaining gap is explicitly justified.
2. **The perishable/durable split** per chapter, per decision 0006, so the
   hardware-independent subset stays visible rather than being reconstructed
   under deadline.

## Licensing and distribution

Full reasoning in `docs/decisions/`. Summary:

| Decision | Choice |
|---|---|
| Repository | Public and complete — not a teaser (0001) |
| Book | Sold in parallel; must genuinely differ from the repo (0001) |
| Code license | Apache-2.0, matching NVIDIA's playbooks (0002) |
| Prose license | CC BY-NC-SA 4.0 (0002) |
| Kindle | Wide distribution; **never KDP Select** — its exclusivity terms are incompatible with a public repo (0003) |
| Contributions | CLA from day one, or open an issue instead (0004) |

Upstream `NVIDIA/dgx-spark-playbooks` is Apache-2.0, so derivation for a
commercial edition is permitted with attribution preserved in `NOTICE`.

**Repository naming.** `operations` is not discoverable. The published repository
should be named for its subject — `dgx-spark-field-manual` or similar — because
"DGX Spark" is the search term that matters. The local directory name is
cosmetic; the GitHub name is not.

## Scope boundaries

In scope: single-Spark operation of the three named workloads, plus supporting
concerns (substrate, memory, thermals, storage, containers, access, uptime).

Out of scope for now, recorded so the boundary is deliberate:

- **Multi-Spark clustering over ConnectX.** Requires a second unit; would be
  permanently UNVERIFIED. The NVIDIA playbook is cloned locally if this changes.
- **Agent and RAG development stacks** (Milvus, workbench). Deprioritized
  relative to the three main workloads.
- **Publishing pipeline** (typesetting, layout, distribution mechanics). The
  manuscript is Markdown; production is a separate project once content exists.

## Success criteria

1. The author opens a runbook during ordinary work without being prompted to.
2. Every command in the corpus is either VERIFIED against a stated version or
   visibly marked otherwise. No unmarked claims.
3. Runbook 90 exists and describes a loop actually followed, such that the
   "everything is Exited" state does not recur silently.
4. `book/OUTLINE.md` gives an honest, current answer to "how much of this book is
   real?"
5. The durable substrate material can be extracted without unpicking it from
   Spark-specific content.
