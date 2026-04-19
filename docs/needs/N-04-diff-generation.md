# N-04: Diff Generation

## Statement

The system shall make it possible, from the archive alone, to obtain the diff
views that shape a reviewer's attention on a pull request: the diff between the
PR's merge base and any round's snapshot, and the diff between any two round
snapshots (not only consecutive rounds).

## Rationale

A reviewer's attention is guided by change. On the first round the reviewer
scans the full PR diff; on every subsequent round they scan what the developer
did since the last pass. Comments in later rounds frequently reference the
inter-round delta implicitly ("you addressed the null check but the naming is
still off"), and the archive cannot recover what "you addressed" refers to
unless that delta is recoverable.

Consecutive-round deltas are the common case, but they are not the only useful
view. A consumer studying a multi-round negotiation may want to compare round 1
against round 4 directly, skipping intermediate rounds that added and reverted
noise. The archive should not force consumers to walk the chain.

This need constrains _what the archive must make possible_, not _how_. The
required diff views could be satisfied by several mechanisms:

- storing precomputed patch files for each pair;
- storing enough snapshots (the merge-base head plus each round's snapshot,
  already captured by [N-03][n03]) that a consumer can compute any diff locally;
- storing a minimal git database with the base commit plus the pushes between
  rounds and tags identifying each round.

Each option has different trade-offs in archive size, external tooling
requirements, and robustness to force-push. Selecting among them is a design
question to be resolved in a later analysis; this need captures what is known
today — that the archive must _enable_ the diff views — without prescribing the
mechanism.

Traces to [W-01 §Diff generation][w01-diff] and to [mission SC #2][sc]
(downstream consumer can reconstruct the full review timeline).

## Acceptance Sketch

The need is met when:

- A consumer reading the archive offline can produce the diff from the PR's
  merge base to any round's snapshot, using only data present in the archive.
- A consumer reading the archive offline can produce the diff from any round's
  snapshot to any other round's snapshot — not only consecutive pairs — using
  only data present in the archive.
- The merge-base commit and each round's snapshot commit are identifiable in the
  archive (by SHA, at minimum) so the diffs are unambiguously named.
- A round whose snapshot is unavailable (force-pushed away, unreachable at
  export time) degrades consistently with the partial-data posture [N-03][n03]
  establishes: diffs that depend on it are explicitly absent rather than
  silently wrong.

[w01-diff]: ../conops/W-01-single-pr-export.md#diff-generation
[sc]: ../mission.md#success-criteria
[n03]: N-03-code-snapshot-preservation.md
