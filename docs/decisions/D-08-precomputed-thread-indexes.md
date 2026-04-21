---
status: accepted
date: 2026-04-21
---

# D-08: Indexes in threads.json are precomputed conveniences

## Context and Problem Statement

Threads are the most access-diverse payload in the archive. Different consumers
want to slice them along different axes: a "render this file" view needs threads
keyed by file path; a "reconstruct this round" view needs threads keyed by
round; a resolution analysis needs threads keyed by `is_resolved`. A single
container shape can only be cheap along one axis; any other grouping forces a
full scan.

The question surfaced at [Turn 10][turn-10] of the 3–4 April 2026 design
conversation, when the target user pushed back on an earlier draft that keyed
`threads.json` by file path: "the threads file should not be keyed by file,
rather have an index that shows different groupings."

## Considered Options

- **Key threads by file path.** `threads["path/to/file"]` gives fast file-scoped
  access; every other grouping requires iteration.
- **Key threads by round.** Round-scoped access is fast; file and resolution
  groupings require iteration.
- **Flat array with precomputed indexes.** Threads live in a single ordered
  list; an `indexes` object alongside it holds secondary maps (`by_file`,
  `by_round`, `by_resolution`) that project the list by each access axis.

## Decision Outcome

Chosen option: _flat array with precomputed indexes_, because privileging any
one axis forces every other consumer to reimplement the same scans the exporter
already has the data to compute once. Indexes are cheap to build at export time
(the exporter already holds the full thread list in memory), trivial to
validate, and free for consumers to use. The flat array keeps the canonical
representation stable while the index set is allowed to grow.

## Consequences

- `threads.json` has two top-level shapes: a flat `threads` array that carries
  the canonical records, and an `indexes` object that projects that array by
  file, by round, and by resolution state. Consumers pick the shape that suits
  their access pattern without re-scanning.
- Schema evolution of the index set is additive. A new index type (e.g.,
  `by_author`) appears alongside the existing ones without breaking consumers
  that already use `by_file` — this matches the schema-evolution principle
  [N-10][n10] asks the archive to satisfy.
- Index entries hold thread IDs, not inlined thread bodies, so the index cost is
  small relative to the thread payload it points into. Consumers de-reference
  IDs against the flat list when they need bodies.
- The flat array remains the source of truth; indexes are a derived view. If the
  two ever disagree, the flat array wins. A consumer that distrusts an index can
  always rebuild it from the list.
- The reference payloads in [reference/03][ref-03] show the `indexes` object
  concretely and document the three index types shipped in v1.

## More Information

- [Original design conversation, Turn 10 — Final corrections][turn-10]
- [Reference/03 — schema specification][ref-03] for the `threads.json` top-level
  shape
- [N-02: Thread Capture][n02] — the need that makes thread access a first-class
  consumer concern
- [N-10: Schema Evolution][n10] — why the index set can grow without breaking
  consumers

[turn-10]: ../reference/00-original-conversation.md#turn-10-final-corrections
[ref-03]: ../reference/03-schema-specification.md
[n02]: ../needs/N-02-thread-capture.md
[n10]: ../needs/N-10-schema-evolution.md
