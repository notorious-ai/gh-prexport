# gh-prexport — API Mapping & Implementation Notes

## GitHub API Endpoints → Schema Fields

This document maps each part of the export schema to the GitHub API endpoint(s)
needed to populate it.

---

### PR Metadata → `pr.json`

| Schema Section            | Endpoint                                   | Notes                                   |
| ------------------------- | ------------------------------------------ | --------------------------------------- |
| `repo.*`                  | `GET /repos/{owner}/{repo}`                | Repo ID, node ID, language, fork status |
| `pr.*`                    | `GET /repos/{owner}/{repo}/pulls/{number}` | All PR fields, author, SHAs, state      |
| `pr.labels`               | Included in PR response                    | Array of label objects                  |
| `participants`            | Inferred from reviews + comments           | Deduplicate across all API responses    |
| `target_user.association` | Included in review/comment responses       | `author_association` field              |

### Rounds → `pr.json.rounds`

Rounds are **constructed, not fetched directly.** The raw data comes from:

| Data                   | Endpoint                                           | Notes                                                                              |
| ---------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Reviews by target user | `GET /repos/{owner}/{repo}/pulls/{number}/reviews` | Filter by `user.login == target_user`. Each submitted review is a candidate round. |
| Review-comment mapping | GraphQL `pullRequest.reviewThreads`                | Associates comments with `pullRequestReview.id`                                    |
| Snapshot SHA per round | `review.commit_id` on each review                  | The SHA the reviewer was looking at                                                |

**Round construction algorithm:**

1. Fetch all reviews by target user, sorted by `submitted_at`.
2. Each review becomes a round (with `review_id`, `state`, `body`).
3. Loose comments (target user comments without a `review_id`) need heuristic
   grouping — likely by timestamp proximity or by the absence of a review
   submission event.
4. `gap_to_next` is computed by examining pushes and reply comments between
   round N's `submitted_at` and round N+1's `submitted_at`.

### Threads → `threads.json`

| Data            | Endpoint                                            | Notes                                                             |
| --------------- | --------------------------------------------------- | ----------------------------------------------------------------- |
| Review threads  | GraphQL: `pullRequest.reviewThreads`                | Provides `isResolved`, `isOutdated`, anchor info, thread grouping |
| Thread comments | GraphQL: `reviewThread.comments`                    | Full conversation within each thread                              |
| Comment details | `GET /repos/{owner}/{repo}/pulls/{number}/comments` | REST fallback — provides `diff_hunk`, `in_reply_to_id`, reactions |

**GraphQL is preferred** for threads because it naturally groups comments into
threads. The REST API returns a flat list of review comments — thread
reconstruction requires following `in_reply_to_id` chains.

**Recommended GraphQL query shape:**

```graphql
query($owner: String!, $repo: String!, $number: Int!, $cursor: String) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $number) {
      reviewThreads(first: 100, after: $cursor) {
        pageInfo { hasNextPage, endCursor }
        nodes {
          id
          isResolved
          isOutdated
          line
          startLine
          diffSide
          originalLine
          originalStartLine
          path
          comments(first: 100) {
            nodes {
              id
              databaseId
              author { login, ... on User { databaseId } }
              authorAssociation
              body
              bodyHTML
              createdAt
              updatedAt
              commit { oid }
              originalCommit { oid }
              diffHunk
              outdated
              replyTo { databaseId }
              pullRequestReview { databaseId, id }
              reactionGroups {
                content
                users { totalCount }
              }
            }
          }
        }
      }
    }
  }
}
```

### Pushes → `pr.json.pushes`

| Data                 | Endpoint                                           | Notes                                                         |
| -------------------- | -------------------------------------------------- | ------------------------------------------------------------- |
| PR commits           | `GET /repos/{owner}/{repo}/pulls/{number}/commits` | Ordered list of commits on the PR branch                      |
| Force push detection | Compare parent SHAs                                | If commit N+1's parent != commit N's SHA, likely a force push |

**Caveat:** After a force push, the pre-force-push commits are no longer in the
PR commit list. The GitHub timeline events API may help detect force pushes:

| Data            | Endpoint                                             | Notes                                                |
| --------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| Timeline events | `GET /repos/{owner}/{repo}/issues/{number}/timeline` | `force-pushed` event type with `before`/`after` SHAs |

### Snapshots → `snapshots/{sha}.json`

| Data                 | Endpoint                                                   | Notes                                       |
| -------------------- | ---------------------------------------------------------- | ------------------------------------------- |
| Files in PR at a SHA | `GET /repos/{owner}/{repo}/pulls/{number}/files`           | Lists changed files with status, patch, SHA |
| File content at SHA  | `GET /repos/{owner}/{repo}/contents/{path}?ref={sha}`      | Returns content + git blob SHA              |
| File content by blob | `GET /repos/{owner}/{repo}/git/blobs/{blob_sha}`           | Raw content, base64 encoded                 |
| Context files        | `GET /repos/{owner}/{repo}/contents/{directory}?ref={sha}` | List directory contents to find siblings    |

**Note:** The PR files endpoint returns files relative to the _current_ head,
not a specific SHA. For per-round snapshots, you need to either:

- Use `GET /repos/{owner}/{repo}/compare/{base}...{sha}` to get the diff at a
  specific point
- Fetch individual file contents at each round's SHA

### Diffs → `diffs/*.patch`

| Data                 | Endpoint                                                                                         | Notes                       |
| -------------------- | ------------------------------------------------------------------------------------------------ | --------------------------- |
| Full PR diff         | `GET /repos/{owner}/{repo}/pulls/{number}` with `Accept: application/vnd.github.v3.diff`         | Returns raw unified diff    |
| Diff at specific SHA | `GET /repos/{owner}/{repo}/compare/{base}...{sha}` with `Accept: application/vnd.github.v3.diff` | Per-snapshot diff from base |
| Inter-round diff     | `GET /repos/{owner}/{repo}/compare/{sha1}...{sha2}` with diff accept header                      | Between two round snapshots |

### Blobs → `blobs/{xx}/{sha}`

| Data         | Endpoint                                         | Notes                       |
| ------------ | ------------------------------------------------ | --------------------------- |
| File content | `GET /repos/{owner}/{repo}/git/blobs/{blob_sha}` | Base64 encoded content      |
| Blob SHA     | Returned by contents API and PR files API        | `sha` field on file objects |

---

## Rate Limiting Considerations

| Concern                                     | Mitigation                                                                |
| ------------------------------------------- | ------------------------------------------------------------------------- |
| REST API: 5,000 requests/hour               | Prefer GraphQL for thread/comment data (single query vs. many REST calls) |
| GraphQL: 5,000 points/hour                  | Batch thread fetching with pagination                                     |
| Blob fetching is N requests per unique file | Cache by blob SHA — skip if already in blob store                         |
| Context files multiply requests             | Scope carefully: same directory/package only, limit depth                 |
| Compare API for diffs                       | One call per snapshot pair — bounded by number of rounds                  |

**Estimated API calls per PR (typical: 3 rounds, 20 threads, 10 files, 30
context files):**

- PR metadata: 1
- Reviews: 1
- Threads (GraphQL): 1-2
- Commits/timeline: 1-2
- Compare/diff per round: 3-6
- File contents for blobs: ~40 (with dedup check)
- Context file directory listings: ~5-10

**Rough total: 50-60 API calls per PR.** At 5,000/hour, that's ~80 PRs/hour
sustained.

---

## Open Implementation Questions

1. **Round detection for loose comments.** Target user sometimes submits
   individual comments without a formal review. How to group these — by
   timestamp window? By commit SHA? Or leave them as "round 0" / unassigned?

2. **Context file scope.** For Go, "same package" is clear (same directory). For
   other languages, the boundary is less obvious. Start with same-directory and
   make it configurable?

3. **Force push handling.** Pre-force-push SHAs may not be fetchable anymore.
   The timeline API records the event, but the actual file contents at old SHAs
   may be gone. Accept this gap or attempt to fetch and gracefully degrade?

4. **Binary files.** Store blob as base64 with `content_encoding: "base64"`, or
   skip entirely? Probably skip with a placeholder noting the file was binary.

5. **Large files.** GitHub API returns content up to 100MB. Files over 1MB use
   the blob API. Need to handle both paths.

6. **PR comments vs review comments.** GitHub has two distinct comment types:
   "issue comments" (on the PR conversation tab) and "review comments" (on the
   diff). The current schema focuses on review comments. Should PR-level
   conversation comments also be captured?
