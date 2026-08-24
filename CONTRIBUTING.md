# Contributing

Corrections are the most valuable thing you can send. This manual's only real
advantage over vendor documentation is that it is true on real hardware, and
that is a claim that decays without help.

## Most useful contributions

1. **"This command doesn't work on my Spark."** Include the exact error, your
   driver and CUDA versions, and the image tag if a container was involved.
   A reproducible failure is worth more than a fix.
2. **Verification on a configuration we don't have.** Much of this manual is
   marked `UNVERIFIED` because it requires hardware or time we lack — a second
   Spark, a multi-hour training run. Confirming or refuting one of those is
   direct progress.
3. **A troubleshooting entry for a failure you actually hit.** Real failures
   only. A troubleshooting section padded with plausible-sounding problems
   nobody has encountered is worse than a short one.

## Status tags are not optional

Every factual claim carries one:

- `VERIFIED` — you ran it. Include the date, the hardware, the relevant versions,
  and real output.
- `UNVERIFIED` — from documentation or reasoning, not tested. Cite the source and
  say why you couldn't test it.
- `BROKEN` — tried, failed. Include the verbatim error and the workaround if you
  found one.

**Never paste an invented terminal transcript.** If the output is not copied from
a real machine, the claim is `UNVERIFIED` and must not show output at all. This
is the one rule whose violation makes the project worthless.

## Contributor License Agreement

By submitting a contribution, you agree to the following:

> You grant the project author a perpetual, worldwide, non-exclusive,
> irrevocable, royalty-free license to reproduce, adapt, publish, translate,
> distribute, and otherwise use your contribution, in any medium and in any
> form, **including commercially** — for example in a print or ebook edition of
> this material.
>
> You retain copyright in your contribution. This grant is in addition to, and
> does not replace, your own rights.
>
> You confirm that the contribution is your own work, that you have the right to
> submit it, and that if you created it in the course of employment you have
> your employer's permission.

**Why this exists.** The repository's prose is CC BY-NC-SA, which does not by
itself let the author include third-party contributions in a commercial edition.
Without this grant, merged contributions would have to be excluded from any
book, or rewritten from scratch. Sorting that out after fifty merged pull
requests is genuinely painful, so it is stated up front.

If you would rather not grant this, **open an issue instead of a pull request**.
A well-described bug report is enormously useful and carries no rights question
at all.

## Placeholder convention

Committed material must not identify a specific machine or person. Readers each
have their own box; a manual full of somebody else's hostnames is noise at best.

| Use | Not |
|---|---|
| `$HOME`, `/home/you` | a real user's home path |
| `<spark-ip>`, or `192.168.1.50` as an obvious example | your actual LAN address |
| `<hostname>`, or `spark` | your actual hostname |
| `enP*` / `wlP*`, noting names vary | a specific interface name |
| `<gpu-bus-id>` | your actual bus ID |

**Sanitizing does not weaken a `VERIFIED` claim.** Replacing identifiers in real
output is expected. Fabricating the output is not. If you sanitize a transcript,
it is still `VERIFIED` — the numbers, versions, and behaviour must remain exactly
what the machine printed.

Hardware specifications, package versions, and measured sizes are **not**
identifying — they are common to the platform — and should be kept. `GB10`,
`121 GiB`, `driver 580.159.03`, and `396 GiB` all stay.

Your own machine's real values belong in `MACHINE.md`, which is gitignored.
Copy `MACHINE.example.md` to start one.

## Practical notes

- One topic per pull request.
- Prose is wrapped at 80 columns.
- Filenames in `troubleshooting/` describe the *symptom*, not the cause — the
  cause is what the reader is trying to find.
