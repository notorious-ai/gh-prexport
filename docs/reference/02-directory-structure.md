# gh-prexport — Directory Structure

## Layout

```
pr-export/
├── version.json
│
└── prs/
    └── {owner}/{repo}/{number}/
        ├── pr.json                     # Metadata, rounds, pushes (the narrative)
        ├── threads.json                # All threads with conversation + indexes
        ├── snapshots/
        │   ├── {sha1}.json             # File manifest for this commit
        │   ├── {sha2}.json
        │   └── ...
        ├── diffs/
        │   ├── base_{sha1}.patch       # Full PR diff at snapshot sha1
        │   ├── base_{sha2}.patch       # Full PR diff at snapshot sha2
        │   ├── {sha1}_{sha2}.patch     # Inter-round diff
        │   └── ...
        └── blobs/
            ├── ab/ab3f7c...            # Content-addressed file storage
            ├── cd/cde891...
            └── ...
```

## File Responsibilities

### `version.json` (root)

Schema version and format metadata. Supports migration tooling.

```json
{
  "schema_version": "0.1.0",
  "min_compatible_version": "0.1.0",
  "format": "single-pr",
  "migrations": []
}
```

- `format`: `"single-pr"` for standalone exports, `"repo-batch"` after merging.
- `migrations`: Array of migration records applied to this export.

### `pr.json` (per PR)

The narrative file — everything about what happened in this PR, without file
contents or full conversations.

Contains:

- Export metadata (who exported, when, target user)
- Repository info (with GitHub IDs)
- PR metadata (title, body, labels, branches, state, SHAs)
- Target user info and role
- Participants list
- Rounds (review sessions with verdicts, thread references, gap-to-next)
- Push timeline (commits between rounds)

Access pattern: Always loaded first. Small regardless of PR size.

### `threads.json` (per PR)

All review threads with full conversation text, plus precomputed indexes.

Contains:

- Flat array of threads, each with anchor, comments, resolution state
- Indexes: by_file, by_round, by_resolution (and extensible)

Access pattern: Loaded when exploring conversations. Potentially large for
heavily-discussed PRs but still text-only.

Future evolution: Can be split into per-file thread files
(`threads/{path}.json`) if needed — the `threads_by_file` index makes this
mechanical.

### `snapshots/{sha}.json` (per commit)

File manifest at a specific commit — lists all PR-affected files and context
files with blob SHA references. No inline content.

Access pattern: Loaded when the user wants to see "the world at round N." Lazy
blob resolution from here.

### `diffs/*.patch` (per snapshot pair)

Raw unified diffs as plain patch files (not JSON). Two types:

- `base_{sha}.patch` — full PR diff from base to this snapshot
- `{from_sha}_{to_sha}.patch` — inter-round diff showing what changed between
  reviews

Access pattern: Loaded for diff visualization. Can be large for big PRs but
stays out of JSON.

### `blobs/{xx}/{full_sha}` (content-addressed)

Raw file contents keyed by git blob SHA. Two-character prefix directory for
filesystem friendliness.

Access pattern: Lazy — loaded only when a specific file at a specific snapshot
is requested. Maximum deduplication within a PR export.

## Design Decisions

### Why split `pr.json` and `threads.json`?

PR metadata + rounds is always small and always needed. Threads scale with
discussion volume and are accessed by specific file/round, not wholesale.
Keeping them separate means `pr.json` stays fast to parse regardless of PR
activity.

### Why rounds live in `pr.json` and not their own file?

Rounds are always small (even 15 rounds is ~30 lines each). They're always
co-loaded with PR metadata. Splitting would create cross-file references without
benefit.

### Why diffs are `.patch` files, not JSON?

Unified diff is a well-understood format with native tool support. They compress
well, render natively in many viewers, and are the largest variable-size
artifact — keeping them out of JSON keeps the JSON files navigable.

### Why blobs are scoped per PR, not shared across repo?

Simplicity for v1. Cross-PR dedup is a merge-command concern. Each PR export is
independently valid and movable.

### Why content-addressed storage?

Git already computes blob SHAs. The API returns them for free. A PR with 5
review rounds touching 3 files in a 20-file package stores ~25 unique blobs
instead of ~100 copies.

## Future: Repo-Batch Format

After running `gh prexport migrate --merge`:

```
pr-export/
├── version.json                        # format: "repo-batch"
├── manifest.json                       # index of all PRs
├── blobs/                              # shared across all PRs in this repo
│   └── ab/ab3f7c...
└── prs/
    └── {owner}/{repo}/
        ├── 142/
        │   ├── pr.json
        │   ├── threads.json
        │   ├── snapshots/
        │   └── diffs/
        ├── 143/
        │   └── ...
        └── ...
```

Key differences: blobs promoted to top level (shared), manifest added for
cross-PR queries, per-PR directories lose their own blob stores.
