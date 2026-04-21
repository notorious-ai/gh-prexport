---
status: accepted
date: 2026-04-21
---

# D-10: Context files scoped to same package/directory

## Context and Problem Statement

A review comment on a single file is rarely understandable on that file alone.
The reviewer's concern lives in a neighbourhood of declarations: types defined
in sibling files, constants the changed code references, helpers it calls into.
The archive therefore has to capture unchanged "context files" alongside the
PR-affected ones. What is too little context collapses downstream replay; what
is too much burns API quota on every round (snapshot fetching is the most
expensive phase of the export). The decision draws the default line.

## Considered Options

- **No context.** Store only PR-affected files. Smallest archive, cheapest API
  usage, poorest comprehensibility for any reviewer or model consuming the
  archive.
- **Same-directory default, language-aware.** For Go repos, context means all
  `.go` files in the directory (i.e., the package). For other languages, the
  same directory is the default. Configurable to widen or narrow.
- **Broader context (imports, transitive).** Follow import edges to pull the
  files a changed file depends on. Most comprehensible, most expensive, and most
  language-dependent to implement robustly.

## Decision Outcome

Chosen option: _same-directory default, language-aware_, because it matches the
unit of meaning in most ecosystems (a directory is a package in Go, a module in
many others) and sets a cost ceiling that scales with the number of rounds
rather than with the repository's import graph. The scope is configurable so
users working in languages or conventions where the default is wrong can widen
or narrow it without editing the tool.

Every context file carries a `reason` field recording why it was included, so
consumers can distinguish same-package defaults from user-configured additions.

## Consequences

- Snapshot fetching is the archive's dominant API cost; the same-directory
  default caps that cost at "directory size × rounds" rather than letting
  import-chasing explode it. This is the efficiency promise [N-07][n07] rests
  on.
- Go gets a natural default because "same directory" and "same package"
  coincide, matching how Go programmers already reason about compilation units.
  Other languages inherit the fallback and pay the cost of its looseness; the
  configurability hatch covers the cases where the fallback misleads.
- The `reason` field makes the context set auditable. A reader of the archive
  can tell which files were pulled in by default and which by configuration,
  which is particularly valuable when the archive is shared with a downstream
  consumer who didn't run the export.
- Context scoping interacts with [N-03][n03] (snapshot preservation): each
  round's snapshot lists both the changed files and the context files with their
  respective blob SHAs. The blob store ([D-04][d-04]) deduplicates across rounds
  automatically, so stable context files pay once even when they appear in every
  snapshot.
- This decision is explicitly the v1 default. A follow-up analysis (planned as
  A-02 in the analysis phase of the SE documentation) will evaluate whether
  language-specific defaults beyond Go should ship with the tool and what a more
  sophisticated context-selection algorithm would offer.

## More Information

- [Reference/04 — API mapping][ref-04] for the context-fetching API pattern
- [N-03: Code Snapshot Preservation][n03] — the need that consumes this decision
- [N-07: API Efficiency][n07] — the cost ceiling same-directory defaults respect
- [D-04: Blobs for file contents, inline for everything else][d-04] — why stable
  context files pay their storage cost only once

[ref-04]: ../reference/04-api-mapping.md
[n03]: ../needs/N-03-code-snapshot-preservation.md
[n07]: ../needs/N-07-api-efficiency.md
[d-04]: D-04-blobs-for-file-contents.md
