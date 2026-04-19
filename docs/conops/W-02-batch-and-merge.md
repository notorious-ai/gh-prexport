# W-02: Batch Export and Merge

Secondary workflows that extend the single-PR export
([W-01](W-01-single-pr-export.md)) toward team-scale data collection. These are
post-MVP but shape the v1 archive format: the `version.json` format
discriminator, per-PR blob scoping, and the directory layout all anticipate
these workflows.

## W-02a: Multi-PR Export

The reviewer exports multiple PRs from one or more repositories in a single
session. Conceptually, this is a loop over W-01: for each PR, the tool runs the
full single-PR export pipeline and writes a self-contained directory under the
shared `prs/` tree.

```
gh prexport notorious-ai/some-project 142 143 145
```

Each PR is exported independently; a failure on one PR does not abort the
others. The blob store remains scoped per PR (no cross-PR deduplication at this
stage), keeping each export self-contained.

The tool reports per-PR summaries and an overall count at the end.

## W-02b: Merge into Repo-Batch Format

After accumulating multiple single-PR exports, the user runs a merge command to
consolidate them into a shared-blob format:

```
gh prexport migrate --merge ./pr-export/
```

This promotes the per-PR blob stores into a single shared blob store at the top
level, adds a `manifest.json` indexing all PRs, and updates `version.json` to
indicate `format: "repo-batch"`. The per-PR directories retain their `pr.json`,
`threads.json`, snapshots, and diffs, but lose their individual blob directories
(blob references now resolve against the shared store).

This workflow matters for the data scientist: batch processing across many PRs
benefits from a single blob store (reduced disk footprint, faster iteration),
and the manifest enables cross-PR queries without scanning the directory tree.

## W-02c: Incremental Re-Export

The reviewer re-exports a PR that has received new activity since the last
export: a new review round, additional comments, or a merge event.

The tool detects the existing export, compares the PR's `updated_at` timestamp
against the export's `exported_at` metadata, and fetches only the delta. New
rounds are appended, new threads added, existing threads updated with new
comments, and new snapshots/blobs fetched. The content-addressed blob store
naturally handles files that haven't changed.

This workflow is the most complex of the three and is likely the last to be
implemented. The v1 archive format supports it (timestamps and IDs are
canonical), but the incremental diffing logic requires careful handling of edge
cases: threads that gained comments, rounds that were dismissed, and pushes that
arrived between the original export and the re-export.

## Relationship to V1

These workflows justify several v1 design decisions that might otherwise appear
premature:

| V1 Decision                    | Workflow it serves             |
| ------------------------------ | ------------------------------ |
| `version.json` format field    | W-02b merge                    |
| Per-PR blob scoping            | W-02a independence             |
| Content-addressed blob storage | W-02c re-export                |
| `exported_at` in metadata      | W-02c delta detection          |
| Directory layout under `prs/`  | W-02a multi-PR and W-02b merge |
