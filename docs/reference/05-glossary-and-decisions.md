# gh-prexport — Glossary & Design Decisions

## Glossary

| Term              | Definition                                                                                                                                                            |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Target user**   | The GitHub user whose review practices are being exported. All data is captured from their perspective.                                                               |
| **Round**         | A single review session — the target user opens the PR at a specific commit, places comments, and optionally submits a formal review with a verdict.                  |
| **Thread**        | A conversation anchored to a code location (file + line), initiated in a specific round. Contains chronologically ordered comments from any participant.              |
| **Snapshot**      | The state of all PR-affected files (and context files) at a specific commit SHA.                                                                                      |
| **Blob**          | A file's content, stored by its git blob SHA for deduplication.                                                                                                       |
| **Context file**  | A file not changed in the PR but stored alongside changed files to provide surrounding context (e.g., other files in the same Go package).                            |
| **Gap**           | The period between two consecutive rounds — includes developer pushes, reply comments, and elapsed time.                                                              |
| **Loose comment** | A review comment submitted by the target user without an associated formal review (e.g., accidentally submitting a single comment instead of batching into a review). |
| **Bronze tier**   | Raw exported data with no analysis, inference, or transformation. Following the Medallion model (bronze → silver → gold).                                             |

## Key Design Decisions

### 1. Threads belong to rounds, comments don't

A thread is born in a round — that's the meaningful association. The
conversation under the thread is timeless; individual comments don't carry round
associations. This avoids misattributing replies and developer responses to
reviewer rounds they weren't part of.

Formalized as [D-01][d-01].

### 2. No inference at export time

Fields like `resolution_type` exist as placeholders (`null`) but are never
populated during export. Thread resolution in practice is messy — people resolve
without fixing, ignore threads, have verbal conversations outside GitHub. The
export captures only what GitHub's API returns (`is_resolved`, `is_outdated`).
Analysis tooling fills in the rest later.

Formalized as [D-02][d-02].

### 3. Data duplication over normalization

The same diff hunk may appear on a thread's `original_diff_hunk`, on each
comment's `diff_hunk_at_time`, and in a patch file under `diffs/`. This is
intentional — each location serves a different access pattern, and consumers
shouldn't need to cross-reference files for basic operations.

Formalized as [D-03][d-03].

### 4. Blobs for file contents, inline for everything else

File contents are the only data externalized to the blob store. Diffs, comments,
metadata, and thread conversations stay in their respective JSON files. The
split is driven by deduplication potential (high for file contents, low for
everything else) and size (file contents dominate storage).

Formalized as [D-04][d-04].

### 5. Patch files are not JSON

Diffs are stored as plain `.patch` files in unified diff format. They're an
established format with native tool support, they compress well, and embedding
large diffs in JSON adds escaping overhead and makes the JSON files hard to
navigate.

Formalized as [D-05][d-05].

### 6. Single-PR export as the atomic unit

Each PR exports to a fully independent directory. No shared state across PRs.
This simplifies the prototype, makes partial exports and retries trivial, and
the merge command handles batching later.

Formalized as [D-06][d-06].

### 7. Both markdown body and HTML body

GitHub renders markdown server-side. The HTML version preserves rendered
suggestion blocks, formatted tables, and other GitHub-specific markdown
extensions that may look different from raw markdown. Both are stored to give
consumers the choice.

### 8. Indexes in threads.json are precomputed conveniences

The `indexes` object in `threads.json` groups thread IDs by file, round, and
resolution status. These are cheap to compute at export time and save consumers
from scanning the full thread list. New index types can be added in future
schema versions without breaking existing data.

### 9. Per-PR blob scope, not cross-repo

Blobs are stored within each PR's directory. Cross-PR deduplication is deferred
to the merge command, which promotes blobs to a shared store when combining
exports into `repo-batch` format. This keeps each export self-contained and
moveable.

### 10. Context files scoped to same package/directory

For Go repos, context means all `.go` files in the same directory (package). For
other languages, same directory is the default. This is the most expensive part
API-wise and should be configurable. The `reason` field on context files
documents why each was included.

[d-01]: ../decisions/D-01-threads-belong-to-rounds.md
[d-02]: ../decisions/D-02-no-inference-at-export-time.md
[d-03]: ../decisions/D-03-data-duplication-over-normalization.md
[d-04]: ../decisions/D-04-blobs-for-file-contents.md
[d-05]: ../decisions/D-05-patch-files-are-not-json.md
[d-06]: ../decisions/D-06-single-pr-atomic-unit.md
