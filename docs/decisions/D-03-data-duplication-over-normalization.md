---
status: accepted
date: 2026-04-21
---

# D-03: Data duplication over normalization

## Context and Problem Statement

Several pieces of data have natural homes in multiple places. A diff hunk is
relevant on the thread it anchors (`original_diff_hunk`), on each individual
comment posted within that thread (`diff_hunk_at_time`), and as a standalone
patch file under `diffs/`. A consumer reading only a thread should know which
hunk the conversation is about; a consumer replaying a round should get the full
diff without opening per-comment payloads; a consumer applying patches should
get a clean `.patch` file. The design must decide whether to inline the data at
each site or normalize it behind a single source of truth with references.

## Considered Options

- **Normalize with references.** Diffs live in one canonical location (the patch
  file, or a keyed registry); threads and comments carry only the key needed to
  join back.
- **Duplicate intentionally.** Each access site carries its own full copy of the
  hunk; consumers never need to cross-reference files for basic operations.

## Decision Outcome

Chosen option: _duplicate intentionally_, because the archive's purpose is to be
read by independent consumers, each with a different access pattern, and
requiring every consumer to implement cross-file joins defeats the
self-containment property the archive is built for. The storage cost is trivial
compared to blob contents; the ergonomic cost of normalization would be paid on
every query.

## Consequences

- Reading a thread in `threads.json` gives a consumer everything it needs about
  that thread — including the diff being discussed — without opening any other
  file. The same applies to per-comment records and to standalone patches.
- This is the structural mechanism behind [N-06: Self-Contained Replay][n06]:
  every in-archive reference resolves inside the archive without a join across
  files being _required_ for basic interpretation.
- Because the export is write-once and immutable, the identical-looking copies
  cannot drift. The "single source of truth" argument for normalization does not
  apply here; the hazard it guards against cannot occur.
- The archive is larger than a normalized form would produce. This is the
  accepted trade: storage is cheap, consumer ergonomics are not.
- Consumers reading the schema must understand that the same hunk appearing in
  multiple locations is intentional. The schema documentation in
  [reference/03][ref-03] names each location explicitly.

## More Information

- [Reference/03 — schema specification][ref-03] for the field-level layout of
  the duplicated hunk sites
- [N-06: Self-Contained Replay][n06] — consumes this decision directly

[ref-03]: ../reference/03-schema-specification.md
[n06]: ../needs/N-06-self-contained-replay.md
