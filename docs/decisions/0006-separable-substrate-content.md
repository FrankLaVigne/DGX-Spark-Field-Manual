# 0006 — Keep substrate content separable from Spark-specific content

**Status:** Accepted · **Date:** 2026-08-24

## Decision

Structure the manual so that hardware-independent material can be lifted out
intact. Tag every section as **perishable** (tied to the DGX Spark) or
**durable** (true of GPU workstations generally).

## Rationale

A hardware-specific book's shelf life is tied to its hardware. The DGX Spark
shipped in late 2025. A successor, a repositioned product line, or a
discontinued one ages the manual quickly and through no fault of its content.

But the two halves age at very different rates:

| Perishable | Durable |
|---|---|
| GB10 specifics, driver 580.x, CUDA 13.0 | systemd, mounts, permissions, service users |
| Spark thermal and power envelope | Unified memory as a concept |
| Exact NGC image tags | aarch64 wheel availability and how to diagnose it |
| Spark storage layout | Docker's effect on the filesystem |
| Two-Spark ConnectX topology | The operational loop — what runs always vs. on demand |

The durable column applies to Jetson, Thor, GB200, and to whatever NVIDIA ships
next — and to the far larger population of practitioners who never buy a Spark
at all. Under decision 0005, it is also the *harder* content to find elsewhere
and therefore the more valuable asset.

The perishable column is what makes the manual immediately useful and findable
today. Both are needed. They should not be entangled.

## Consequence for the substrate runbook

`05-linux-for-ai-people.md` is promoted from supporting material to a
first-class component, written as though it may one day stand alone — because it
may. "Linux for people who know AI but not Linux" is plausibly a larger and
longer-lived title than anything Spark-specific, and it is the piece least
threatened by a hardware refresh.

Practically: it should not assume the reader has a Spark. Spark-specific
illustrations appear as examples, clearly marked, not as premises.

## Consequence for structure

- Each runbook states its scope at the top: Spark-specific, or general.
- Examples drawn from `proteus` are labelled as examples, so a reader on
  different hardware knows what to substitute.
- `book/OUTLINE.md` tracks the perishable/durable split per chapter, so the
  extractable subset is always visible rather than being reconstructed under
  deadline.

## Cost

Some duplication and some awkwardness. A concept that is 80% general with a
Spark-specific wrinkle has to be written in two registers. Accepted — the
alternative is discovering the entanglement at the moment the extraction is
needed, which is exactly when there is no time for it.

## Revisit if

- The Spark line is discontinued, which triggers the extraction rather than a
  reconsideration.
- The durable material grows large enough to warrant splitting into its own
  repository sooner than planned.
