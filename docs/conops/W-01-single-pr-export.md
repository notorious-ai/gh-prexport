# W-01: Single-PR Export

The primary workflow. The reviewer exports one pull request they participated
in, producing a self-contained archive that captures the full review timeline.

## Preconditions

- The reviewer has `gh` installed and authenticated.
- The target PR exists and is readable by the authenticated user.
- The reviewer was a participant (reviewer, author, or both) on the PR.

## Narrative

The reviewer identifies a PR they want to export, typically by its URL or by
owner/repo and number. They invoke the CLI:

```
gh prexport notorious-ai/some-project 142
```

The tool begins by fetching PR metadata: title, body, state, branches, labels,
and participants. It identifies the target user (defaulting to the authenticated
user) and determines their role on this PR.

### Round construction

The tool fetches all reviews submitted by the target user, sorted
chronologically. Each submitted review becomes a candidate round: the
`commit_id` on the review is the snapshot SHA (the code the reviewer was looking
at), the review body and state are the verdict, and the `submitted_at` timestamp
anchors the round in the timeline.

Loose comments (comments the target user submitted without a formal review,
often by accident) are detected and grouped with their nearest round or flagged
as unassigned. The grouping heuristic is an open analysis item
([A-01](../analysis/)).

For each consecutive pair of rounds, the tool computes the gap: the elapsed
time, the commits pushed by the developer between rounds, and the number of
reply comments in that interval. The gap captures the developer's response to
the reviewer's feedback.

### Thread capture

Using the GraphQL API, the tool fetches all review threads on the PR. Each
thread is a conversation anchored to a specific file and line, initiated in a
specific round. The thread carries GitHub's resolution state (`is_resolved`,
`is_outdated`) but no inference about how or why it was resolved.

Threads are mapped to rounds by their originating review: the first comment in
the thread references a `pullRequestReview.id` that links to a round. Threads
initiated by someone other than the target user have a null round association.

The full conversation within each thread is captured chronologically: the
reviewer's initial comment, the developer's reply, any follow-up exchanges. This
conversation arc (which may span multiple rounds) preserves the negotiation
pattern: objection, explanation, softened position, acceptance with conditions.

### Snapshot assembly

For each round's snapshot SHA, the tool builds a file manifest: every file
changed in the PR at that point, plus context files (unchanged files in the same
directory or package). Each file entry records its path, language, change
status, and a reference to its content-addressed blob.

### Blob fetching

File contents are fetched by git blob SHA. Before fetching, the tool checks
whether the blob already exists in the export's blob store (from a previous
snapshot in the same PR). Content-addressed storage means a file that hasn't
changed between rounds is fetched once and referenced from multiple snapshots.

### Diff generation

For each snapshot, the tool fetches two diffs:

- The full PR diff from the merge base to this snapshot
- The inter-round diff from the previous snapshot (if any)

Diffs are stored as plain `.patch` files in unified diff format.

### Push timeline

The tool fetches the PR's commit list and timeline events to reconstruct the
push history: which commits appeared between rounds, whether any were
force-pushes, and how the code evolved between the reviewer's sessions.

### Archive assembly

The tool writes the complete archive to the local filesystem: `version.json` at
the root, then the PR directory containing `pr.json` (metadata, rounds, pushes),
`threads.json` (all threads with indexes), snapshot manifests, diff patches, and
the blob store.

The tool reports a summary: rounds found, threads captured, blobs stored, total
API calls made.

## Error Scenarios

### Rate limit exhaustion

The tool may hit REST or GraphQL rate limits mid-export. The expected behavior
is to report the limit, indicate what was partially written, and exit. The
content-addressed blob store means a re-run skips already-fetched blobs, making
re-export after rate limit recovery relatively cheap.

### Unreachable snapshot SHA

If a round's snapshot SHA is unreachable (e.g., after a force-push), the tool
cannot fetch file contents or diffs for that snapshot. The round is still
recorded in `pr.json` with its review metadata, but the corresponding snapshot
manifest is either omitted or marked as unreachable. This is a graceful
degradation; the export continues with available data.

### No reviews by target user

If the target user has no reviews on the PR, the export contains PR metadata,
participants, and push timeline, but no rounds. This is a valid (if sparse)
export.

### Binary or oversized files

Files exceeding the API size limit or detected as binary are recorded in the
snapshot manifest with appropriate status but no blob content. The export is not
interrupted by individual file fetch failures.

## Resumability

The v1 export is not resumable mid-run. If the process is interrupted, the
partial output may be incomplete. However, re-running the export for the same PR
is idempotent at the blob level: content-addressed blobs that were already
written are skipped on re-fetch, and the JSON files are regenerated from
scratch.

True incremental export (detecting new activity since the last export and
appending to an existing archive) is a future workflow
([W-02](W-02-batch-and-merge.md)).
