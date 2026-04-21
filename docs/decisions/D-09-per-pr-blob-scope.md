---
status: accepted
date: 2026-04-21
---

# D-09: Per-PR blob scope, not cross-repo

## Context and Problem Statement

Once the archive externalises file bytes to a content-addressed blob store
([D-04][d-04]), the next question is where that store lives. A repo-wide blob
store shared by all PRs maximises deduplication: a file present in ten PRs is
stored once. A per-PR blob store forfeits that deduplication but keeps each PR
archive self-contained — the PR directory can be copied, shared, or deleted
without touching anything outside it. The choice has to cooperate with the
per-PR atomicity the single-PR export decision already commits to.

## Considered Options

- **Shared repo-wide blob store from day one.** One blob directory for the whole
  repository; every PR export writes into and reads from it. Cross-PR
  deduplication is automatic.
- **Per-PR blob store; merge command promotes blobs later.** Each PR's archive
  holds its own blob store under its own directory. A later merge step (W-02b)
  promotes blobs to a shared store when combining archives into `repo-batch`
  format.

## Decision Outcome

Chosen option: _per-PR blob store_, because it keeps per-PR atomicity honest.
[D-06][d-06] commits to each PR archive being independent — movable,
restartable, deletable on its own — and a shared blob store would either break
that property or require coordination with state outside the PR directory. The
storage cost of duplicated blobs is accepted as the v1 concession, reclaimed at
merge time rather than paid up front.

## Consequences

- Every PR archive is truly self-contained. Moving, archiving, or deleting a PR
  directory does not affect any other PR. The
  [self-containment success criterion][sc] holds at the PR boundary without
  caveats.
- A file that appears in ten PRs is stored ten times in v1. The cost is bounded
  by how similar the affected files are across PRs and is recovered by
  [W-02b: merge into repo-batch format][w02b], which promotes blobs to a shared
  store once archives are combined.
- The merge command's value is sharpened by this decision: without cross-PR blob
  scope in v1, deduplication _is_ what W-02b provides, not a detail of it. That
  clarifies the v1-to-batch transition.
- Consumers can reason about blob lifetimes one PR at a time. There is no global
  garbage collection concern, no cross-archive reference tracking, no migration
  path to worry about when a single PR is re-exported.

## More Information

- [D-04: Blobs for file contents, inline for everything else][d-04] — the
  decision that establishes the blob store this one scopes
- [D-06: Single-PR export as the atomic unit][d-06] — the parent decision this
  one makes operational for blob storage
- [W-02b: merge into repo-batch format][w02b] — where cross-PR deduplication
  lands
- [Mission success criterion 4][sc] — the self-containment property per-PR scope
  preserves

[d-04]: D-04-blobs-for-file-contents.md
[d-06]: D-06-single-pr-atomic-unit.md
[w02b]: ../conops/W-02-batch-and-merge.md#w-02b-merge-into-repo-batch-format
[sc]: ../mission.md#success-criteria
