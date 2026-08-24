# 0003 — Publish KDP wide, never KDP Select

**Status:** Accepted · **Date:** 2026-08-24

## Decision

If a Kindle edition ships, enrol it in standard KDP **wide** distribution.
Do not enrol in **KDP Select** under any circumstances.

## Rationale

KDP Select requires digital exclusivity: the book's content may not be available
free elsewhere for the duration of enrolment. That is flatly incompatible with
decision 0001, which puts the entire manual in a public repository.

This is not a tradeoff to weigh. Enrolling in Select while the repository exists
would breach the terms.

## What is given up

- **Kindle Unlimited page-read revenue.** Meaningful in high-volume consumer
  fiction; negligible for a niche technical title whose readers buy deliberately.
- **Countdown Deals and Free Book Promotions.** Promotional mechanics aimed at
  impulse purchase, largely irrelevant to a professional audience arriving from
  a search result.
- **70% royalty in certain territories** (India, Brazil, Japan and others) that
  Select otherwise unlocks. Real, but small against a modest expected volume.

Every item forfeited is a revenue optimisation. Goal ranking says revenue loses
to reach.

## What is gained

Wide distribution reaches Apple Books, Kobo, Google Play Books, and — importantly
for a technical title — library and institutional channels via Draft2Digital or
similar. Technical readers skew away from Kindle relative to general fiction.

## Related risk: price-matching

Amazon reserves the right to price-match a title to $0 where identical content is
freely available. Mitigated by decision 0001's requirement that the two editions
genuinely differ: the repository holds reference documentation, the book is an
edited, sequenced narrative. This should be treated as a real constraint on the
book's production, not a formality.

## Revisit if

- Amazon changes the Select exclusivity terms.
- The repository is withdrawn (which would mean revisiting 0001 first).
