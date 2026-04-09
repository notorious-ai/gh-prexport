# gh-prexport — Schema Specification

Schema version: `0.1.0`

---

## `pr.json`

```jsonc
{
  "schema_version": "0.1.0",

  "export_metadata": {
    "exported_at": "2026-04-04T12:00:00Z", // ISO 8601
    "exported_by": "gh-prexport",
    "exporter_version": "0.1.0",
    "target_user": "yourusername",
    "target_user_id": 111222
  },

  "repo": {
    "owner": "someorg",
    "owner_id": 555666,
    "name": "someproject",
    "repo_id": 777888,
    "github_node_id": "R_abc...",
    "primary_language": "go",
    "is_fork": false,
    "default_branch": "main"
  },

  "pr": {
    "number": 142,
    "pr_id": 1234567890, // GitHub numeric ID
    "github_node_id": "PR_xyz...", // GitHub GraphQL node ID
    "url": "https://github.com/someorg/someproject/pull/142",
    "api_url": "https://api.github.com/repos/someorg/someproject/pulls/142",
    "title": "Fix connection pool cleanup on dial failure",
    "body": "## Summary\n\nFixes a connection leak...",
    "state": "merged", // open | closed | merged
    "author": "junior-dev",
    "author_id": 333444,
    "branch_head": "fix/pool-cleanup",
    "branch_base": "main",
    "base_sha": "fff000...",
    "head_sha": "abc999...",
    "labels": [
      {
        "name": "bugfix",
        "id": 9001
      }
    ],
    "created_at": "2026-03-01T10:00:00Z",
    "updated_at": "2026-03-05T14:30:00Z",
    "merged_at": "2026-03-05T14:30:00Z", // nullable
    "closed_at": "2026-03-05T14:30:00Z", // nullable
    "merge_commit_sha": "merge123...", // nullable
    "final_files_changed": 12,
    "final_additions": 340,
    "final_deletions": 87
  },

  "target_user": {
    "login": "yourusername",
    "user_id": 111222,
    "github_node_id": "U_abc...",
    "role": "reviewer", // author | reviewer | both
    "association": "MEMBER" // OWNER | MEMBER | CONTRIBUTOR | etc.
  },

  "participants": [
    {
      "login": "junior-dev",
      "user_id": 333444,
      "github_node_id": "U_def...",
      "role": "author", // author | reviewer | commenter
      "association": "CONTRIBUTOR"
    },
    {
      "login": "other-reviewer",
      "user_id": 555666,
      "github_node_id": "U_ghi...",
      "role": "reviewer",
      "association": "MEMBER"
    }
  ],

  "rounds": [
    {
      "round_number": 1, // 1-indexed, chronological
      "snapshot_sha": "abc123...", // -> snapshots/{sha}.json
      "submitted_at": "2026-03-02T09:00:00Z",

      "review": { // nullable — null for loose comments
        "review_id": 98765432,
        "github_node_id": "PRR_abc...",
        "state": "CHANGES_REQUESTED", // APPROVED | CHANGES_REQUESTED | COMMENTED | DISMISSED
        "body": "Couple of things to address before this is ready.",
        "body_html": "<p>Couple of things to address before this is ready.</p>"
      },

      "thread_ids": ["PRT_abc123", "PRT_abc124", "PRT_abc125"],

      "gap_to_next": { // nullable — null for last round
        "duration_seconds": 86400,
        "push_shas": ["def456...", "def789..."], // -> pushes[]
        "developer_reply_count": 3
      }
    },
    {
      "round_number": 2,
      "snapshot_sha": "ghi789...",
      "submitted_at": "2026-03-03T11:00:00Z",

      "review": {
        "review_id": 98765500,
        "github_node_id": "PRR_def...",
        "state": "APPROVED",
        "body": "Looks good. Just rename that helper, I trust you.",
        "body_html": "<p>Looks good. Just rename that helper, I trust you.</p>"
      },

      "thread_ids": ["PRT_abc130"],

      "gap_to_next": null
    }
  ],

  "pushes": [
    {
      "sha": "abc123...",
      "parent_sha": "fff000...",
      "message": "initial implementation",
      "author": "junior-dev",
      "author_id": 333444,
      "timestamp": "2026-03-01T11:00:00Z",
      "is_force_push": false, // detected by parent chain break
      "after_round": null // nullable — null if before first review
    },
    {
      "sha": "def456...",
      "parent_sha": "abc123...",
      "message": "address review: fix dial leak",
      "author": "junior-dev",
      "author_id": 333444,
      "timestamp": "2026-03-02T15:00:00Z",
      "is_force_push": false,
      "after_round": 1
    },
    {
      "sha": "def789...",
      "parent_sha": "def456...",
      "message": "address review: add test",
      "author": "junior-dev",
      "author_id": 333444,
      "timestamp": "2026-03-02T16:00:00Z",
      "is_force_push": false,
      "after_round": 1
    }
  ]
}
```

---

## `threads.json`

```jsonc
{
  "threads": [
    {
      "thread_id": "PRT_abc123",
      "github_node_id": "PRRT_abc123...",

      // Which review round initiated this thread
      "round_number": 1, // nullable — null if not part
      //   of a formal round (loose comment,
      //   or started by someone else)

      "anchor": {
        "path": "pool/conn.go",
        "line": 47, // nullable
        "start_line": 45, // nullable — for multi-line comments
        "side": "RIGHT", // LEFT (deletion) | RIGHT (addition)
        "original_commit": "abc123..."
      },

      // The diff hunk at thread creation, with extra context lines
      "original_diff_hunk": "@@ -38,12 +38,20 @@ func (p *Pool) Dial(ctx context.Context) (*Conn, error) {\n ...",
      "original_diff_hunk_context_lines": 8,

      // Raw GitHub state — no inference
      "is_resolved": true,
      "is_outdated": false,

      // Placeholder for analysis layer — always null at export time
      "resolution_type": null, // future: addressed | deferred | conceded | abandoned

      "comments": [
        {
          "comment_id": 12345678, // GitHub numeric ID
          "github_node_id": "PRRC_xyz...",
          "github_review_id": 98765432, // nullable — which review this was part of
          "author": "yourusername",
          "author_id": 111222,
          "is_target_user": true,
          "association": "MEMBER",
          "body": "This leaks the connection if `Dial` returns an error after the socket is opened. You need a deferred cleanup here.",
          "body_html": "<p>This leaks the connection if <code>Dial</code> returns an error after the socket is opened. You need a deferred cleanup here.</p>",
          "created_at": "2026-03-02T09:05:00Z",
          "updated_at": null, // nullable
          "commit_sha": "abc123...",
          "diff_hunk_at_time": "@@ -38,12 +38,20 @@ ...",
          "is_outdated": false,
          "in_reply_to_id": null, // nullable — numeric comment ID
          "reactions": {} // map[string]int — emoji name -> count
        },
        {
          "comment_id": 12345679,
          "github_node_id": "PRRC_xyz2...",
          "github_review_id": null,
          "author": "junior-dev",
          "author_id": 333444,
          "is_target_user": false,
          "association": "CONTRIBUTOR",
          "body": "Good catch, fixed in def456. Added `defer conn.Close()` before the error check.",
          "body_html": "...",
          "created_at": "2026-03-02T14:30:00Z",
          "updated_at": null,
          "commit_sha": "abc123...",
          "diff_hunk_at_time": "@@ -38,12 +38,20 @@ ...",
          "is_outdated": false,
          "in_reply_to_id": 12345678,
          "reactions": { "thumbs_up": 1 }
        }
      ]
    }
  ],

  // Precomputed indexes — thread_id references into threads[]
  "indexes": {
    "by_file": {
      "pool/conn.go": ["PRT_abc123", "PRT_abc124"],
      "pool/pool.go": ["PRT_abc125"]
    },
    "by_round": {
      "1": ["PRT_abc123", "PRT_abc124", "PRT_abc125"],
      "2": ["PRT_abc130"]
    },
    "by_resolution": {
      "resolved": ["PRT_abc123", "PRT_abc125"],
      "unresolved": ["PRT_abc124", "PRT_abc130"],
      "outdated": []
    }
  }
}
```

---

## `snapshots/{sha}.json`

```jsonc
{
  "sha": "abc123...",
  "timestamp": "2026-03-01T11:00:00Z",

  // Every file that is part of the PR at this point
  // (i.e., differs from base)
  "files": [
    {
      "path": "pool/conn.go",
      "language": "go",
      "status": "modified", // added | modified | deleted | renamed
      "previous_path": null, // nullable — populated for renames
      "blob_sha": "ab3f7c...", // -> blobs/ab/ab3f7c...
      "content_encoding": "utf8", // utf8 | base64 (for binary detection)
      "size_bytes": 4821
    },
    {
      "path": "pool/conn_test.go",
      "language": "go",
      "status": "added",
      "previous_path": null,
      "blob_sha": "ff2e1a...",
      "content_encoding": "utf8",
      "size_bytes": 2105
    }
  ],

  // Unchanged files that provide context
  // Same package/directory as changed files
  "context_files": [
    {
      "path": "pool/pool.go",
      "language": "go",
      "blob_sha": "ee91b2...",
      "reason": "same_package" // same_package | same_directory | imported_by
    },
    {
      "path": "pool/types.go",
      "language": "go",
      "blob_sha": "cc44d1...",
      "reason": "same_package"
    }
  ],

  // Precomputed diffs for convenience
  // These reference the patch files in diffs/
  "diffs": {
    "from_base": "base_abc123.patch", // -> diffs/base_abc123.patch
    "from_previous_snapshot": null // nullable — null for first snapshot
    //   otherwise -> diffs/{prev}_{this}.patch
  }
}
```

---

## `diffs/*.patch`

Plain unified diff files. Not JSON. Two naming conventions:

- **`base_{sha}.patch`** — Full PR diff from merge-base to this snapshot.
- **`{from_sha}_{to_sha}.patch`** — Inter-round diff between two snapshots.

Example content:

```diff
diff --git a/pool/conn.go b/pool/conn.go
index ab3f7c1..ee91b20 100644
--- a/pool/conn.go
+++ b/pool/conn.go
@@ -38,12 +38,20 @@ func (p *Pool) Dial(ctx context.Context) (*Conn, error) {
     conn, err := net.DialContext(ctx, "tcp", p.addr)
     if err != nil {
         return nil, fmt.Errorf("dial: %w", err)
     }
+    defer func() {
+        if err != nil {
+            conn.Close()
+        }
+    }()
```

---

## `blobs/{xx}/{full_sha}`

Raw file contents. No wrapping, no JSON, no metadata — just the file bytes.

Directory structure uses first two hex characters of the SHA as a prefix
directory for filesystem friendliness (same convention as git's object store).

Example: blob SHA `ab3f7c9e...` is stored at `blobs/ab/ab3f7c9e...`

---

## `version.json`

```jsonc
{
  "schema_version": "0.1.0",
  "min_compatible_version": "0.1.0",
  "format": "single-pr", // single-pr | repo-batch
  "migrations": [] // array of migration records
}
```

---

## Type Reference

### Enums

| Field                 | Values                                                                                                                |
| --------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `target_user.role`    | `author` · `reviewer` · `both`                                                                                        |
| `participant.role`    | `author` · `reviewer` · `commenter`                                                                                   |
| `association`         | `OWNER` · `MEMBER` · `COLLABORATOR` · `CONTRIBUTOR` · `FIRST_TIME_CONTRIBUTOR` · `FIRST_TIMER` · `MANNEQUIN` · `NONE` |
| `pr.state`            | `open` · `closed` · `merged`                                                                                          |
| `review.state`        | `APPROVED` · `CHANGES_REQUESTED` · `COMMENTED` · `DISMISSED`                                                          |
| `anchor.side`         | `LEFT` · `RIGHT`                                                                                                      |
| `file.status`         | `added` · `modified` · `deleted` · `renamed`                                                                          |
| `content_encoding`    | `utf8` · `base64`                                                                                                     |
| `context_file.reason` | `same_package` · `same_directory` · `imported_by`                                                                     |
| `resolution_type`     | `addressed` · `deferred` · `conceded` · `abandoned` · `null`                                                          |
| `version.format`      | `single-pr` · `repo-batch`                                                                                            |

### Nullable Fields

Fields marked `nullable` use JSON `null` when absent. Key nullable fields:

- `pr.merged_at`, `pr.closed_at`, `pr.merge_commit_sha`
- `round.review` (loose comments without formal review submission)
- `round.gap_to_next` (last round)
- `thread.round_number` (threads not initiated by target user)
- `comment.github_review_id` (standalone reply comments)
- `comment.updated_at`, `comment.in_reply_to_id`
- `push.after_round` (pushes before first review)
- `snapshot.diffs.from_previous_snapshot` (first snapshot)
- `thread.resolution_type` (always null at export, analysis fills later)
