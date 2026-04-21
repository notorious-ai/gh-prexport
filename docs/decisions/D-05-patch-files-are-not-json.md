---
status: accepted
date: 2026-04-21
---

# D-05: Patch files are not JSON

## Context and Problem Statement

The archive contains diffs of several sizes and shapes: small hunks quoted as
context on a single thread, round-level diffs spanning a handful of files, and
full between-snapshot diffs that can run into thousands of lines. Every diff has
an established native representation (unified format) that existing tools
understand. The archive must decide whether to embed diffs as strings inside
JSON documents or to store them as standalone `.patch` files alongside the JSON.

## Considered Options

- **Embed in JSON.** Every diff is a string field on the JSON record that
  contains it; the archive has one storage surface fewer to document.
- **Standalone `.patch` files.** Diffs are written to their own files under
  `diffs/` in unified format; JSON records reference them by path.

## Decision Outcome

Chosen option: _standalone `.patch` files_, because the unified diff format is
already the lingua franca for diffs and pushing it through JSON escaping breaks
everything that expects to read patches with standard tooling. `git apply`,
`patch`, IDE diff viewers, and code-review tools all consume `.patch` files
directly; requiring consumers to unescape JSON strings before using any of them
is a cost paid on every interaction.

Embedded diffs also degrade the JSON files that carry them. A single
thousand-line diff turns a navigable JSON document into one with a single
multi-kilobyte string field, which `jq`, editors, and version-control diffing
handle poorly.

## Consequences

- Diffs compress well and remain human-readable in the archive. A reader (or
  reviewer, or tool) can `cat` a `.patch` file and understand it without
  interpretation.
- The archive has two storage surfaces besides blobs — JSON and patches — but
  the split is sharply defined by file extension, so consumers know where to
  look.
- Thread and comment records can still carry short hunk context inline (per
  [D-03][d-03]) because those excerpts are small, navigable, and pattern-matched
  rather than tool-applied. The full patch lives in `diffs/` for consumers that
  need to apply or render it.
- Diffs are not content-addressed because they are comparatively small and do
  not deduplicate the way file contents do. The blob/patch split therefore
  matches what each format is best at: blobs for dedupable bytes, patches for
  tool-native diffs.

## More Information

- [Reference/02 — directory structure][ref-02] for the on-disk layout of
  `diffs/`
- [D-03: Data duplication over normalization][d-03] — why short hunks still
  appear inline inside thread and comment records
- [D-04: Blobs for file contents, inline for everything else][d-04] — the
  sibling format-matching decision for file bytes

[ref-02]: ../reference/02-directory-structure.md
[d-03]: D-03-data-duplication-over-normalization.md
[d-04]: D-04-blobs-for-file-contents.md
