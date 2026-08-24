# DGX Spark Field Manual

Operating notes for the NVIDIA DGX Spark, written by someone using one.

Vendor documentation tells you what the hardware can do. This tells you what
happens when you try.

---

## Start here

```
$ du -sh ~/.ollama
16K     /home/you/.ollama            <-- looks empty

$ ollama list
llama4:maverick    244 GB
gpt-oss:120b        65 GB
deepseek-r1:70b     42 GB
...

$ sudo du -sh /usr/share/ollama/.ollama
396G    /usr/share/ollama/.ollama    <-- a quarter of the drive
```

396 GiB, invisible from the home directory. Three things conspire to hide it:
Ollama runs as a **systemd system service**, so its home is `/usr/share/ollama`
rather than yours; that directory is mode `750` owned by the `ollama` account;
and `du` fails quietly rather than loudly when it cannot read a directory.

Not one of those is a machine learning fact. If you can explain a KV cache but
have never had reason to learn what a service account is, there was no path by
which you were going to know this — and the cost of not knowing was a quarter of
a terabyte and an unexplained full disk.

That gap is what this manual is for.

## Who this is for

Two kinds of people buy this hardware, and they arrive from opposite directions.

| | You know | You need explained |
|---|---|---|
| **AI practitioner** | the model | the machine |
| **Home labber** | the machine | the model |

If you are an AI practitioner, this manual **will not** explain LoRA,
quantization, attention, or checkpointing. You know those better than we do.
It **will** explain systemd, mounts, permissions, `$PATH`, `journalctl`, and
what Docker is doing to your filesystem — the invisible prerequisites nobody
mentions.

If you came from home lab and self-hosting, systemd is not news. Machine
learning concepts appear as short marked asides you can read or skip:

> **Domain aside — skip this if you work with models daily.**
> Quantization reduces the numeric precision of a model's weights…

The procedures are the same for both of you. Only the explanations differ.

There is also material vendor documentation omits entirely because it assumes a
datacenter: power draw, fan noise, where to physically put the thing, what it
costs to leave running, and how it coexists with a NAS and a home network.

## Every claim carries a status

The only real advantage this has over vendor documentation is that it is true on
actual hardware. That claim is worthless unless it is auditable, so nothing is
unmarked:

> **VERIFIED** — 2026-08-24, driver 580.159.03, CUDA 13.0
> Run on real hardware. Output below is copied from a terminal, not composed.

> **UNVERIFIED** — from NVIDIA playbook, untested here (requires a second Spark)
> Reasoning or documentation, not experience. No output is shown, because
> there is none to show.

> **BROKEN** — 2026-08-24. Fails with `Bus error (core dumped)`.
> Tried. Did not work. Workaround, if any, is stated.

**No invented terminal output. Ever.** If a transcript appears, a machine
printed it. Identifiers like hostnames and LAN addresses are replaced with
placeholders — sanitizing output does not make a claim less verified;
fabricating it makes the whole project worthless.

`UNVERIFIED` is not an apology. It is the backlog, and it will be tracked in
`book/OUTLINE.md` so you can always see how much of this is real.

## Layout

| Directory | Open it when |
|---|---|
| `runbooks/` | You are working. Task-oriented: "I want to serve a model." |
| `troubleshooting/` | You are stuck. Symptom-indexed — grep it for your error string. |
| `book/` | You want the whole thing in order. |
| `docs/decisions/` | You want to know why this project is arranged the way it is. |

Filenames in `troubleshooting/` describe the **symptom**, not the cause. The
cause is what you are trying to find.

Your own machine's specifics — hostname, addresses, versions — belong in
`MACHINE.md`, which is gitignored. Copy
[`MACHINE.example.md`](MACHINE.example.md) to start one. It doubles as a drift
log, which is what you will want the first time something breaks after an
update.

## Status: early

Honest accounting, since the whole premise is not overstating things.

**Written:** project design, licensing, and seven decision records.

**Not yet written:** every runbook, every troubleshooting entry, and the book
outline. The directories above are the plan, not the contents.

If you found this expecting a finished manual, it is not one yet.

## Contributing

Corrections are the most valuable thing you can send, especially "this command
does not work on my Spark" with the exact error and your driver version. A
reproducible failure is worth more than a fix.

See [`CONTRIBUTING.md`](CONTRIBUTING.md). It includes a contributor license
grant; if you would rather not agree to it, **open an issue instead of a pull
request** — a good bug report is often more useful anyway.

## License

This repository is **dual-licensed**. GitHub's sidebar may show nothing, because
a split license does not match its detector. It is not unlicensed.

| Content | License |
|---|---|
| Code, scripts, configuration | [Apache-2.0](LICENSE-code) |
| Prose — runbooks, chapters, troubleshooting | [CC BY-NC-SA 4.0](LICENSE-prose) |

Share and adapt the prose freely, with credit, under the same terms, for
non-commercial use. The NonCommercial term exists to deter republishing this as
a padded ebook — not to obstruct teaching, talks, study groups, or internal use
at work. For commercial use, open an issue; it is a conversation, not a refusal.

See [`LICENSE`](LICENSE) and
[decision 0002](docs/decisions/0002-dual-license-apache-and-cc-by-nc-sa.md).

---

Portions derive from [NVIDIA/dgx-spark-playbooks](https://github.com/NVIDIA/dgx-spark-playbooks)
(Apache-2.0); see [`NOTICE`](NOTICE). NVIDIA, DGX, and CUDA are trademarks of
NVIDIA Corporation. This project is not affiliated with, endorsed by, or
sponsored by NVIDIA.
