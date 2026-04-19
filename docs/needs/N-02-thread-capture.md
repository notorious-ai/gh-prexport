# N-02: Thread Capture

## Statement

The system shall capture every review thread on the pull request as an anchored,
chronologically ordered conversation: the location in code the thread discusses,
the round in which it was born, the full sequence of comments exchanged by any
participant, and the resolution state GitHub records for it.

## Rationale

Threads are where the substantive back-and-forth of a review lives: the initial
objection, the developer's reply, the softened position, the acceptance with
conditions. A training or evaluation dataset that omits this arc loses the
reasoning structure the archive exists to preserve.

Two properties of threads shape this need and distinguish it from N-01:

- A thread belongs to the round in which it was initiated, not to the comments
  within it. The conversation under a thread is timeless and may continue across
  later rounds without the thread "moving." ([D-01][d-01])
- The archive records only what GitHub knows (`is_resolved`, `is_outdated`) and
  draws no inference about whether a resolved thread was actually addressed,
  ignored, or resolved out-of-band. Thread realism — people crap all over
  threads — is a property of the source data, not something the exporter tries
  to clean up. ([D-02][d-02])

Traces to [W-01 §Thread capture][w01-thread] and to [mission SC #2][sc]
(downstream consumer can reconstruct the full review timeline: who said what, on
which code, in what order).

## Acceptance Sketch

The need is met when, for a given PR:

- Every review thread the PR carries appears in the archive, regardless of which
  participant initiated it.
- Each thread records the code location it discusses — file path, line position,
  and the commit SHA the comment was originally placed on — so the thread
  remains interpretable even after later force-pushes rewrite branch history.
- Each thread records the originating round, or a null origin when the initiator
  is not the target user.
- Each thread contains its comments in chronological order, with the full body
  of every comment and the author of each comment.
- Each thread carries GitHub's resolution signals verbatim; the archive contains
  no field that infers _why_ the thread was resolved, ignored, or left open.

[d-01]: ../reference/05-glossary-and-decisions.md#1-threads-belong-to-rounds-comments-dont
[d-02]: ../reference/05-glossary-and-decisions.md#2-no-inference-at-export-time
[w01-thread]: ../conops/W-01-single-pr-export.md#thread-capture
[sc]: ../mission.md#success-criteria
