# Operational Environment

## Execution Context

gh-prexport runs as a GitHub CLI extension on the developer's local machine. The
runtime is a single Go binary invoked via `gh prexport`, inheriting the `gh`
CLI's authentication and configuration. There is no server component, no
persistent daemon, and no background process; each invocation is a
self-contained export run that writes to the local filesystem and exits.

The tool targets the same platforms as the `gh` CLI: macOS, Linux, and Windows.
It assumes a POSIX-like filesystem for the content-addressed blob store
(two-character prefix directories, SHA-based filenames).

## Network Context

All data acquisition flows through the GitHub API, accessed via `gh api`. Two
API surfaces are used:

- **REST API** for PR metadata, reviews, commits, file contents, diffs, and
  timeline events. Rate-limited to 5,000 authenticated requests per hour.
- **GraphQL API** for review threads and their comments. Rate-limited to 5,000
  points per hour (each query costs variable points based on complexity).

The tool must function over standard HTTPS; no special network configuration is
required beyond what `gh auth` provides. Export throughput is bounded by API
rate limits, not bandwidth.

**Estimated cost per PR**: 50-60 API calls for a typical PR with 3 rounds, 20
threads, 10 changed files, and 30 context files. At sustained throughput,
approximately 80 PRs can be exported per hour.

## Data Context

### Repository access

The tool can export any PR that the authenticated `gh` user can read. This
includes public repositories, private repositories the user belongs to, and
organization repositories with appropriate permissions. Repository visibility
determines what data is accessible, not what the tool attempts to fetch.

### PR states

PRs exist in three states (open, closed, merged) and all are valid export
targets. The richest exports come from merged PRs with multiple review rounds,
but the tool must handle:

- PRs with no reviews by the target user (no rounds to construct)
- PRs in draft state (may have incomplete review activity)
- PRs closed without merge (the review arc may be truncated)
- Very old PRs where some historical data may be unavailable

### Data permanence

Not all GitHub data is permanent:

- **Force-pushed SHAs** may become unreachable. File contents at pre-force-push
  commits cannot always be fetched, though the timeline API records the event.
- **Deleted branches** lose their ref but commits remain accessible by SHA as
  long as they are reachable in the repository's object graph.
- **Deleted repositories** make all data inaccessible.
- **User renames** are handled transparently by GitHub's API via numeric IDs and
  node IDs.

## Storage Context

The export writes to the local filesystem as a directory tree rooted at a
user-specified output path. The structure follows [reference/02][ref-02]:

```
pr-export/
├── version.json
└── prs/{owner}/{repo}/{number}/
    ├── pr.json
    ├── threads.json
    ├── snapshots/{sha}.json
    ├── diffs/*.patch
    └── blobs/{xx}/{sha}
```

Each PR export is self-contained and independently moveable. Blob deduplication
occurs within a single PR export (files unchanged across snapshots are stored
once); cross-PR deduplication is deferred to a future merge command.

Storage requirements scale with the number of unique files across review rounds,
not with the number of rounds themselves (content-addressed storage absorbs
duplication). A typical PR export is expected to be in the low megabytes range.

## Constraints

| Constraint             | Bound               | Impact                                                |
| ---------------------- | ------------------- | ----------------------------------------------------- |
| REST rate limit        | 5,000 requests/hour | Bounds throughput; blob fetching dominates call count |
| GraphQL rate limit     | 5,000 points/hour   | Bounds thread/comment fetching                        |
| File content API limit | 100 MB per file     | Files above this threshold cannot be fetched          |
| Blob API threshold     | 1 MB                | Files over 1 MB require the blob API endpoint         |
| Force-push data loss   | Unpredictable       | Pre-force-push file contents may be unrecoverable     |
| GraphQL pagination     | 100 items per page  | Thread-heavy PRs require multiple round-trips         |

## Assumptions

1. The user has `gh` installed and authenticated with sufficient permissions to
   read the target repository.
2. The target PR has review activity by the target user (otherwise the export
   contains metadata but no rounds).
3. The local filesystem has sufficient space for the export (typically single
   digit MB per PR).
4. Network connectivity to `api.github.com` is available during the export run.

[ref-02]: ../reference/02-directory-structure.md
