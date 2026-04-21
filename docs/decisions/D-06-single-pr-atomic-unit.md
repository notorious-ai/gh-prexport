---
status: accepted
date: 2026-04-21
---

# D-06: Single-PR export as the atomic unit

## Context and Problem Statement

The long-term goal is to capture a team's review practices across many PRs, but
every aggregated export starts from individual PRs. Two paths are natural: treat
the archive as repo-level from day one (one big directory, shared blob store,
shared thread index), or treat each PR as its own independent archive and add
batching later. The choice shapes how the prototype ships, how failures
propagate, and how later workflows evolve.

The target user settled this at [Turns 7–8][turns-7-8] of the 3–4 April 2026
design conversation, in the same exchange that established the
directory-tree-with-content-addressed-blobs format: "start with one independent
directory per PR, add a merge command later."

## Considered Options

- **Repo-level atomicity from the start.** One archive per repository; shared
  blob store and shared indexes span all PRs. Cross-PR deduplication comes for
  free, but every export coordinates with the shared state.
- **Per-PR atomicity.** Each PR exports to a fully independent directory. No
  shared state across PRs. Cross-PR deduplication deferred to a merge command.

## Decision Outcome

Chosen option: _per-PR atomicity_, because the prototype needs to ship one PR at
a time, and a failure on PR 37 should not invalidate the work already done on
PRs 1–36. Independence makes partial exports, retries, and cherry-picking
individual PR archives trivial. The efficiency concerns that motivate repo-level
atomicity (shared blob store, repo-wide indexes) can be reclaimed later by a
merge step without paying for them upfront.

## Consequences

- The v1 output is a self-contained directory per PR. This is what
  [W-02a: multi-PR export][w02a] operates on — a simple loop over PRs, each
  producing an independent archive, with no coordination between invocations.
- [W-02b: merge into repo-batch format][w02b] is the planned path to reclaim
  cross-PR deduplication and shared indexes. Its design can proceed on its own
  schedule because nothing about v1 commits to a final aggregated shape.
- Per-PR atomicity pulls the blob store inside each PR's directory as well, so
  the archive travels as a unit. That corollary has its own record as the
  per-PR-blob-scope decision.
- Initial storage is less efficient than a repo-level archive would be (a file
  present in ten PRs is stored ten times in v1). The cost is accepted as a
  prototype concession, addressable by merge.
- Several fields in the v1 schema (`version.json` format tag, `exported_at`
  timestamp, directory layout under `prs/`) are motivated by future batch
  workflows even though v1 exports single PRs. The
  [relationship-to-v1 table in W-02][w02-v1] lists them explicitly.

## More Information

- [Original design conversation, Turns 7–8 — Format and the medallion
  model][turns-7-8]
- [W-02: Batch and Merge][w02] — the workflow family that consumes this
  decision, with [W-02a][w02a] relying on per-PR atomicity and [W-02b][w02b]
  introducing merge
- [Relationship to V1 table in W-02][w02-v1] — concrete list of v1 choices
  motivated by batch workflows

[turns-7-8]: ../reference/00-original-conversation.md#turn-7-8-format-and-the-medallion-model
[w02]: ../conops/W-02-batch-and-merge.md
[w02a]: ../conops/W-02-batch-and-merge.md#w-02a-multi-pr-export
[w02b]: ../conops/W-02-batch-and-merge.md#w-02b-merge-into-repo-batch-format
[w02-v1]: ../conops/W-02-batch-and-merge.md#relationship-to-v1
