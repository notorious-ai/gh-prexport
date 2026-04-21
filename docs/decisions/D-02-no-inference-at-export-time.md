---
status: accepted
date: 2026-04-21
---

# D-02: No inference at export time

## Context and Problem Statement

Several GitHub-side signals in a pull request are structurally present but
semantically thin. `is_resolved` on a thread says a button was clicked;
`is_outdated` says a diff position no longer maps onto HEAD. Neither tells the
consumer why — whether the concern was fixed, deferred, agreed away, or resolved
verbally outside GitHub. Comment intent (suggestion, nit, blocking question) is
similarly absent from the raw API. The export must decide whether to classify
these signals at capture time or to stop at the raw surface.

The question surfaced in the 3–4 April 2026 design conversation at
[Turn 5][turn-5]. The initial draft introduced a `resolution_type` field
populated by heuristic inference; the target user pushed back with "people crap
all over [threads]", arguing that thread resolution is too noisy to classify
honestly from the API alone.

## Considered Options

- **Infer at export.** The exporter runs heuristics against resolution, comment
  phrasing, and diff state to populate semantic fields such as `resolution_type`
  and comment intent labels.
- **Capture only what the API returns; reserve inference fields as null
  placeholders.** The schema keeps fields like `resolution_type` so a later
  analysis layer can fill them, but the exporter never populates them.

## Decision Outcome

Chosen option: _capture only what the API returns, with null placeholders_,
because inference at export time would bake the exporter's heuristics into the
archive and make every consumer inherit them. Thread resolution semantics depend
on context the API cannot see — verbal conversations, follow-up PRs, a
reviewer's habit of resolving-and-forgetting — and any classification will be
wrong often enough to corrupt the dataset it is meant to seed.

Placeholders remain in the schema so downstream analysis tools have idiomatic
slots to write into; the boundary between raw capture and analysis is drawn at
the export tool's edge.

## Consequences

- The archive is faithful to what GitHub knows. Mission scope explicitly
  excludes "analysis, inference, or scoring of review quality" and "resolution
  inference"; this decision is what makes that boundary enforceable at the data
  layer (see [mission scope][mission-scope]).
- The same philosophy generalises to [graceful degradation][n08]: when data is
  missing, the archive records the absence rather than guessing the missing
  value. Honesty about what is and is not known is a property of the whole
  bronze tier.
- Consumers that want resolution classification, comment intent labels, or
  similar enrichment must build a silver-tier transformation. The archive gives
  them stable inputs; it does not give them the labels.
- Schema evolution can add new placeholder fields without breaking existing
  consumers, because the rule "placeholders are null at export" is the invariant
  rather than the specific field list.

## More Information

- [Original design conversation, Turn 5 — Thread realism][turn-5]
- [N-02: Thread Capture][n02] — consumes this decision for thread resolution
  handling
- [N-08: Graceful Degradation][n08] — shares the "record absence, do not guess"
  principle
- [Mission scope][mission-scope] — classification and inference are out of scope

[turn-5]: ../reference/00-original-conversation.md#turn-5-thread-realism
[n02]: ../needs/N-02-thread-capture.md
[n08]: ../needs/N-08-graceful-degradation.md
[mission-scope]: ../mission.md#scope
