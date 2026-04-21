# N-01: Review Round Reconstruction

## Statement

The system shall reconstruct the sequence of review rounds conducted by the
target user on a pull request, where each round represents a single review
session anchored to the commit SHA the reviewer was looking at, the batch of
feedback they produced, and the verdict (or absence of one) with which they
concluded the session.

## Rationale

Rounds are the organising unit of the export. Every other captured artifact —
threads, snapshots, diffs, gaps — is defined relative to a round, so the
fidelity of round reconstruction governs the fidelity of the archive as a whole.
Rounds are not supplied directly by the GitHub API; they are
[constructed][round-construction] from reviews, comments, and timeline events.
This constructive step is where the temporal structure of a real review session
is recovered from the flat data GitHub exposes, and is the reason a bespoke
export tool is needed at all.

Traces to [W-01 §Round construction][w01-round] as the primary workflow step,
and to [mission SC #3][sc] ("round construction accurately reflects the actual
review sessions as they happened").

## Acceptance Sketch

The need is met when, for a given PR and target user, the export contains an
ordered sequence of rounds in which:

- Each round carries the snapshot SHA the reviewer was looking at, the review
  verdict (or an explicit marker that none was submitted), and the
  `submitted_at` timestamp that anchors it on the timeline.
- The ordering of rounds matches the chronological order of the target user's
  review submissions.
- Feedback the target user authored outside a formal review (loose comments) is
  either attributed to a round or explicitly recorded as unassigned — never
  silently dropped.
- A reviewer who submitted no reviews yields zero rounds without failing the
  export.

The choice of mechanism for loose-comment attribution is deferred to
[A-01 Loose Comment Grouping][a-01] and is not part of this need.

[round-construction]: ../glossary.md#domain-terms
[w01-round]: ../conops/W-01-single-pr-export.md#round-construction
[sc]: ../mission.md#success-criteria
[a-01]: ../analysis/
