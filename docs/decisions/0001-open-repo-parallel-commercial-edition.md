# 0001 — Open repository with a parallel commercial edition

**Status:** Accepted · **Date:** 2026-08-24

## Decision

Publish the full manual in a public repository, free and complete. Sell a
curated edition in parallel on Kindle and print.

The repository is not a teaser. It is the whole thing.

## Rationale

The instinct that free content cannibalizes paid sales is not supported by the
record in technical publishing. *Pro Git*, *The Rust Programming Language*,
*Automate the Boring Stuff with Python*, and *Eloquent JavaScript* are all
available free and complete online, and all sell well in print and ebook —
several as category bestsellers.

The mechanism is that free and paid serve different moments:

| Free repository | Paid edition |
|---|---|
| Discovery, search ranking, linkability | Convenience, offline, curation |
| "Does this answer my question right now?" | "I want the whole thing, edited" |
| Credential a hiring manager can inspect | Patronage — supporting the author |

Readers who get value from the free version are the *population most likely to
buy*, not a population lost to the sale.

Given the goal ranking in `README.md`, the repository is also the more important
artifact on its own terms. Prospective employers, collaborators, and NVIDIA
staff can read it without buying anything. A book behind a paywall is invisible
to exactly the people whose attention is the point.

## The two editions must genuinely differ

Not merely as strategy — as a safeguard. Amazon may price-match a paid title to
$0 if it detects identical content available free.

- **Repository** — reference material. Task-indexed, symptom-indexed, current.
- **Book** — a book. Edited, sequenced, with narrative connective tissue and
  the reasoning that a reference doc omits.

This is a better product on both sides. Reference docs make bad books, and books
make frustrating reference docs.

## Alternatives considered

**Closed, book only.** Maximizes per-reader revenue on a small audience while
forfeiting the primary goal. Rejected on the operative rule.

**Free sample chapters, paid remainder.** The worst of both — too little to rank
in search or establish credibility, and readers resent the wall. Rejected.

**Free only, no book.** Loses the "published author" credential, which carries
real weight for speaking submissions and consulting, at a low marginal cost once
the content exists. Rejected.

## Revisit if

- The free version demonstrably suppresses sales (measurable, unlikely).
- The audience turns out to be overwhelmingly corporate, where procurement
  prefers a purchasable artifact and free access is irrelevant.
- Ecosystem exposure stops being the primary goal.
