# 0007 — Two reader profiles, and how the home labber is served

**Status:** Accepted · **Date:** 2026-08-24
**Amends:** [0005](0005-audience-expert-in-model-novice-in-machine.md)

## Decision

The manual serves two reader profiles that approach the same hardware from
opposite directions:

| Profile | Knows | Needs explained |
|---|---|---|
| **A — AI practitioner** (primary) | the model | the machine |
| **B — Home labber** (secondary) | the machine | the model |

**Profile A remains primary.** Profile B is served two ways:

1. **Additive content** — power, noise, placement, always-on economics, LAN
   integration, backups. This conflicts with nothing and is a genuine gap.
2. **Labeled, skippable domain asides** — never inline prose.

## The conflict this resolves

Decision 0005 established a rule: *never explain LoRA, always explain systemd.*

For a home labber that rule is precisely backwards. Someone running Proxmox with
opinions about VLANs finds systemd unremarkable; what they do not know is what
quantization costs, why a 70B model will not fit in the memory they have, or
what a KV cache is holding.

0005 is not wrong — it is under-specified. It described one reader as though
they were the only one.

## Why serve B at all

**Market.** The self-hosting and home lab communities are substantially larger
than the population of DGX Spark owners who arrived via an AI-practitioner
career. A machine marketed as a personal AI supercomputer, sitting on a desk at
home, draws that audience by construction.

**Durability.** Under 0006, home lab material is durable: power, noise,
networking, and uptime survive every hardware refresh. It ages better than
anything Spark-specific.

**The author is partly this reader.** The reference machine runs Traefik,
Open WebUI, Milvus, ArangoDB, MinIO, and xrdp. That is a home lab stack, not a
workstation configuration. Writing it is not an act of imagination.

**It is an unserved gap.** Vendor documentation assumes a datacenter: rack power,
managed cooling, an IT function, a network somebody else runs. None of those
assumptions hold in a spare room.

## Mechanism

The two explanation types are treated **asymmetrically**, following from A being
primary:

- **Substrate explanations** (systemd, mounts, permissions) are **inline main
  text**. Profile A needs them to follow the procedure at all.
- **Domain explanations** (quantization, KV cache, model sizing) are **marked
  asides**. Profile A skips them on sight.

Asides use a consistent, scannable marker:

> **Domain aside — skip this if you work with models daily.**
> Quantization reduces the numeric precision of a model's weights…

The procedures themselves are identical for both readers. Only the explanation
around them differs, which is what makes a single manual viable rather than two.

## Content added for Profile B

Additive, no conflict with existing scope:

- **Power, noise, heat, placement** — it lives in a room with people in it
- **Always-on economics** — idle draw times electricity rate; 24/7 vs. on-demand
- **LAN integration** — reverse proxy, DNS, VPN, mDNS, certificates
- **Sharing the endpoint** — family, other machines, other services
- **Backups and UPS behaviour** — there is no IT department
- **Coexistence** — slotting a Spark beside a NAS, Proxmox host, or Home
  Assistant install

## The honest risk

**Serving two audiences can serve neither.** A manual that stops every few
paragraphs to explain something half its readers already know is exhausting for
both halves.

The entire mitigation is labelling discipline. An aside must be visually
skippable at a glance, never longer than a short paragraph, and never load-
bearing — if a procedure cannot be followed without reading the aside, the aside
is misclassified and belongs in the main text.

If asides start growing, or start being required, that is the signal that this
decision has failed and Profile B needs its own document.

## Alternatives considered

**Profile A only.** Simplest, preserves 0005 unmodified, forfeits the larger
audience and an unserved niche. Rejected — the additive content costs little and
the reach matters more (see the goal ranking in `README.md`).

**Two separate books.** Cleanest voice for each, roughly doubles the work, and
splits an already-small audience. Premature at this stage; remains the fallback
if asides fail.

**Full dual-track, both audiences co-primary.** Every concept explained from both
directions. This is the version that serves neither, and produces a manual twice
as long and half as usable. Rejected.

## Revisit if

- Asides grow beyond a short paragraph, or become load-bearing.
- Reader feedback shows one profile is being poorly served.
- Profile B turns out to dominate, which would invert the primary/secondary
  ranking rather than abandon the structure.
