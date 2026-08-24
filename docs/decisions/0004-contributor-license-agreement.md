# 0004 — Require a CLA from day one

**Status:** Accepted · **Date:** 2026-08-24

## Decision

Every contribution carries a license grant to the author permitting commercial
use, stated in `CONTRIBUTING.md` and agreed by the act of submitting.
Contributors who decline are directed to open issues rather than pull requests.

## Rationale

This is the decision most often missed, and it is close to irreversible if
missed.

When someone contributes prose to a CC BY-NC-SA repository, that contribution is
licensed to the project under those same terms. **CC BY-NC-SA does not grant the
author the right to relicense it into a commercial edition.** The contributor
retains copyright and has granted only non-commercial rights.

The consequence, discovered late, is ugly: publish a book containing fifty
merged community contributions and every one is a rights defect. Remedies are
tracking down each contributor for retroactive permission — with people who have
changed jobs, addresses, or interest — or excising and rewriting every merged
contribution. Both are expensive; neither is reliable.

Stated up front, the cost is one paragraph.

## Design

Deliberately lightweight — no signing service, no bot, no account linkage.
Submitting a pull request constitutes agreement, as stated in `CONTRIBUTING.md`.
This is the model used by many small projects and is proportionate to the scale
here. Heavier machinery would deter exactly the drive-by corrections that are
most valuable.

The grant is non-exclusive and contributors retain their own copyright. They are
not signing anything away; they are adding a permission.

## The escape hatch matters

Some contributors object to CLAs on principle, and that position is reasonable —
it grants a private party commercial rights to donated work. Rather than argue,
`CONTRIBUTING.md` routes them to issues.

This costs the project almost nothing. A precise bug report — "this command
fails on driver 580.159.03 with this exact error" — is frequently *more*
valuable than a patch, and carries no rights question whatsoever.

## Alternatives considered

**No CLA, rewrite contributions.** Every merged contribution must be
independently reworded before it can enter the book. Error-prone, and the
distinction between "rewritten" and "derivative" is a poor thing to be
adjudicating about your own book.

**DCO instead of a CLA.** The Developer Certificate of Origin certifies
provenance — that you have the right to submit — but grants no additional
license beyond the project's. It does not solve this problem.

**Accept issues only, no pull requests.** Fully avoids the problem and forfeits
most community contribution. Disproportionate.

## Revisit if

- The project moves to a foundation or acquires co-authors.
- Contribution volume justifies proper CLA tooling.
- The commercial edition is abandoned, at which point the grant is unnecessary.
