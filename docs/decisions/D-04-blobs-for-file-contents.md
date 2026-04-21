---
status: accepted
date: 2026-04-21
---

# D-04: Blobs for file contents, inline for everything else

## Context and Problem Statement

The archive carries data with very different storage characteristics. File
contents are bulky and repeat across snapshots: a single unchanged file may
appear in five rounds with identical bytes. Thread conversations, comments,
round metadata, and indexes are comparatively tiny and do not repeat — every
comment is unique to its thread. A blanket policy of "externalize everything
large" or "keep everything inline" ignores this split and either fragments small
data into blobs or forces large data through JSON.

## Considered Options

- **Inline everything.** File contents, diffs, and metadata all live inside JSON
  documents.
- **Externalize file contents only.** File contents go to a content-addressed
  blob store; diffs, comments, metadata, and thread conversations stay in their
  respective JSON files.
- **Externalize anything potentially large.** File contents, large diffs, and
  long comment bodies are all blobbed.

## Decision Outcome

Chosen option: _externalize file contents only_, because the split follows the
data. File contents dominate archive storage and deduplicate naturally — the
same file appears across multiple snapshots and gets stored once when keyed by
git blob SHA. Everything else is small, non-deduplicable, and more useful inline
where consumers already expect to find it.

Diffs are not blobs either; they are plain `.patch` files on disk. That choice
is made by the next decision in this series, which addresses their specific
tool-compatibility concerns.

## Consequences

- The blob store is the archive's only content-addressed surface. Every other
  file is navigable by path and readable as-is. This keeps the mental model
  simple: JSON for structure, blobs for file bytes, patches for diffs.
- File-content deduplication is automatic and scales with how much code stays
  stable across rounds. The "blob dominates storage" assumption only strengthens
  as the number of rounds grows.
- [N-03: Code Snapshot Preservation][n03] gets the efficient path it needs
  without pulling every snapshot into JSON. Snapshot manifests reference blob
  SHAs; consumers fetch by SHA when they need bytes.
- Inline JSON stays navigable with standard tools (`jq`, editors, text search).
  A 2 MB thread document is always a 2 MB thread document, not a shell of
  pointers into blob storage.
- Consumers must understand two storage surfaces instead of one. The
  blob-vs-inline boundary is documented in [reference/02][ref-02] and hinges on
  a single criterion: file bytes live in blobs, everything else lives inline.

## More Information

- [Original design conversation, Turns 7–8 — Format and the medallion
  model][turns-7-8]
- [Reference/02 — directory structure][ref-02] for the on-disk layout of the
  blob store
- [N-03: Code Snapshot Preservation][n03] — consumer of this decision

[turns-7-8]: ../reference/00-original-conversation.md#turn-7-8-format-and-the-medallion-model
[ref-02]: ../reference/02-directory-structure.md
[n03]: ../needs/N-03-code-snapshot-preservation.md
