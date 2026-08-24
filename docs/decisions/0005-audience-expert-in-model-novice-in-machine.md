# 0005 — Audience: expert in the model, novice in the machine

**Status:** Accepted · **Date:** 2026-08-24

## Decision

The reader is a data scientist or AI engineer — frequently from the Windows
ecosystem — who is expert in machine learning and a beginner at Linux systems
administration.

This inverts the manual's explanation policy:

- **Never explain** LoRA, quantization, attention, batch size, KV cache,
  checkpointing, learning rate schedules. The reader knows these better than the
  author does. Explaining them is condescending and wastes the page.
- **Always explain** systemd, mounts, `$PATH`, file permissions, service users,
  `journalctl`, what Docker does to the filesystem, why a command needs `sudo`.
  These are invisible prerequisites nobody told the reader about.

## Rationale

This combination — deep domain expertise, shallow substrate knowledge — is
common among AI practitioners and is served by no existing document.

| Document type | Assumes | Explains |
|---|---|---|
| Linux / sysadmin documentation | You know the substrate | The domain |
| AI tutorials and vendor playbooks | You know the domain | Nothing — `sudo docker run --gpus all --ipc=host`, good luck |
| **This manual** | You know the domain | **The substrate** |

Nobody writes for the person who can explain a KV cache but has never heard of
systemd. That person has just spent several thousand dollars on hardware they
cannot fully reach, and there is good reason to believe they are a large share
of DGX Spark buyers — the machine is marketed to AI practitioners, not to
systems administrators.

## The founding evidence

Discovered on the reference machine while drafting, and worth recording
because it is the thesis in miniature:

```
$ du -sh ~/.ollama
16K     /home/you/.ollama            <-- appears empty

$ ollama list
llama4:maverick    244 GB
llama4:scout        67 GB
gpt-oss:120b        65 GB
deepseek-r1:70b     42 GB
llama3:latest      4.7 GB

$ sudo du -sh /usr/share/ollama/.ollama
396G    /usr/share/ollama/.ollama    <-- 24% of the drive
```

396 GiB — a quarter of the disk — invisible from the home directory. Three
Linux facts conspire to hide it:

1. Ollama runs as a **systemd system service**, under a service account named
   `ollama`.
2. A service account's home is therefore `/usr/share/ollama`, not `$HOME`.
3. That directory is mode `750`, owned by `ollama`, so the human user cannot
   read it — and `du` fails quietly rather than loudly.

Not one of those is an AI fact. There is no path by which domain expertise
produces this knowledge, and the cost of not having it is 396 GiB of an
unexplained full disk.

A fourth, smaller trap sits on top: `ollama list` totals ~423 GB while `du`
reports 396 G. Both are correct. `du -sh` reports **GiB** and Ollama reports
**GB**; 396 GiB is 425 GB. A reader who notices the discrepancy and assumes data
corruption has been misled by units alone.

A related observation from the same machine: every container started by hand
with `docker run` was found stopped, some for ten months, while `ollama.service`
was still running — because the installer registered it with systemd. On Windows
the reader would have made it a Service and never thought about it again. Same
concept, different name, and nobody supplied the name.

## Consequences

1. **Voice.** Command-first remains, but no command appears unexplained: what it
   does, what correct output looks like, what to do when it differs.
2. **A substrate thread**, not a primer chapter. Concepts are explained inline,
   attached to the command that requires them, because nobody reads a Linux
   primer front to back. `runbooks/05-linux-for-ai-people.md` collects only what
   genuinely recurs.
3. **GUI parity.** Where LM Studio, ComfyUI, JupyterLab, VS Code Remote, or RDP
   is the better tool, say so plainly. The author reaches this machine over
   `xrdp` from a Windows box; that is a legitimate access pattern the vendor
   documentation never acknowledges. The terminal is not a hazing ritual.
4. **The verification policy becomes more important, not less.** A reader who
   cannot yet distinguish a correct command from a plausible one is entirely
   dependent on the manual being right.

## Alternatives considered

**General Linux/AI practitioner audience.** Broader, and served by everything
already. Rejected — an undifferentiated book competes with the whole field.

**Assume CLI fluency (the original spec).** Written before the author's own
background was clear. It would have reproduced the exact gap the manual exists
to close.

## Revisit if

- Reader feedback shows the substrate explanations are unnecessary.
- The Spark's tooling matures enough that the substrate stops leaking through.
