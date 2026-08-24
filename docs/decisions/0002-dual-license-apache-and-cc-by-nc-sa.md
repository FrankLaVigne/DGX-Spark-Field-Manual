# 0002 — Dual license: Apache-2.0 for code, CC BY-NC-SA 4.0 for prose

**Status:** Accepted · **Date:** 2026-08-24

## Decision

| Content | License |
|---|---|
| Code, scripts, configuration | Apache License 2.0 |
| Prose, runbooks, chapters | CC BY-NC-SA 4.0 |

Canonical texts are in `LICENSE-code` and `LICENSE-prose`. Neither was
reproduced from memory: the Apache text was copied from NVIDIA's own repository
on disk, and the CC text was fetched from creativecommons.org.

## Rationale

**Apache-2.0 for code.** It matches what NVIDIA uses for `dgx-spark-playbooks`,
so derived scripts stay license-compatible with their upstream. It carries an
express patent grant, which MIT lacks, and corporate legal teams approve it
without discussion — relevant when the audience is professionals who may want to
use these scripts at work.

**CC BY-NC-SA 4.0 for prose.** The NonCommercial clause addresses one specific,
active threat: scraping open technical writing, padding it with generated text,
and reselling it on Kindle. This is common enough to be an industry nuisance and
would directly undercut decision 0001. NC provides a clean basis for takedown.

ShareAlike ensures adaptations stay open, which serves the courseware goal.

## The upstream question

NVIDIA's playbooks are Apache-2.0, which permits commercial use and derivation
provided attribution and `NOTICE` survive. **There is therefore no licensing
obstacle to a commercial edition built partly on that material.** Attribution is
recorded in `NOTICE` and at each point of use.

## The honest tradeoff

NC is not an OSI-approved open source license, and some of the open community
objects to it on principle. More practically, NC makes *corporate* reuse legally
awkward — including by NVIDIA, whose attention is the goal. A DevRel team that
wants to quote the manual in an official channel hits a clause requiring a
conversation.

Accepted anyway, for two reasons. The author is the sole copyright holder and
can grant any additional license on request, instantly and at no cost. And
"contact me for commercial licensing" is a business-development funnel rather
than a barrier — it manufactures precisely the conversation the project exists
to start.

If NC ever measurably impedes ecosystem adoption, that is a signal to revisit,
and relicensing to CC BY-SA is a one-commit change.

## Alternatives considered

**CC BY-SA for prose.** More standard, genuinely open, Wikipedia's license. But
ShareAlike alone does not deter Kindle republishing — violators ignore the
obligation, and enforcement burden falls on the author either way. NC gives a
clearer, cheaper claim.

**CC BY.** Maximum reach, zero protection. Reconsider if the republishing threat
proves overstated.

**All rights reserved.** Contradicts the courseware goal. Rejected.

## Revisit if

- NVIDIA or another ecosystem partner is blocked by NC from amplifying the work.
- The Kindle republishing threat proves overstated in practice.
- The project acquires co-authors, at which point sole-copyright flexibility ends.
