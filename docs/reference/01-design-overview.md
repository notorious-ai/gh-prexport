# gh-prexport — Design Overview

## Purpose

A GitHub CLI extension (`gh prexport`) that exports a user's pull request review
interactions — comments, reviews, threads, code snapshots, and diffs — into a
structured, self-contained archive. The exported data serves as a
**bronze-tier** dataset (borrowing from the Medallion model) for later
processing, analysis, and fine-tuning LLM agents to replicate the user's code
review practices, standards, and craftsmanship.

## Philosophy

- **Export, not analysis.** This tool captures raw data. Processing, insights,
  and resolution inference belong to future tooling.
- **Self-contained replay.** The export should be sufficient to build a
  visualization playground that replays past PRs entirely from the exported data
  — no GitHub API calls needed at render time.
- **Data duplication is acceptable.** Convenience and self-containment outweigh
  storage efficiency at the bronze tier.
- **Capture what GitHub knows, nothing more.** No inference about intent,
  resolution reasons, or sentiment. Placeholder fields (e.g., `resolution_type`)
  are included for future analysis layers but exported as `null`.

## Core Concepts

### Rounds

The fundamental unit of a review session. A round represents: a code snapshot
(HEAD at the time the reviewer opened the PR) + the batch of comments submitted
together + the verdict (or absence of one).

Between rounds, developers push commits and reply to threads. This inter-round
activity is captured as `gap_to_next` on each round.

### Threads

A thread is a conversation anchored to a code location, born in a specific
round. The conversation under a thread is timeless — comments don't shift to new
rounds. The round is the thread's origin, not a per-comment attribute.

Threads reflect messy reality: people resolve without fixing, ignore threads,
have verbal conversations outside GitHub. The export captures only what GitHub
knows (`is_resolved`, `is_outdated`) and leaves interpretation to the analysis
layer.

### Snapshots

The world at a specific commit SHA — every file that is part of the PR at that
point, plus unchanged context files (e.g., same Go package). File contents are
stored as content-addressed blobs for deduplication.

### Blobs

File contents stored by git blob SHA. Natural deduplication — the same file
appearing unchanged across multiple snapshots is stored once.

## Technology

- **Language:** Go
- **Distribution:** GitHub CLI extension (`gh extension install ...`)
- **API:** GitHub REST + GraphQL APIs, accessed via `gh` auth layer
- **Output format:** Directory tree with content-addressable blob store (see
  `02-directory-structure.md`)

## Versioning & Evolution

- Each export carries a `version.json` with schema version and migration
  metadata.
- Initial format is `single-pr` — one self-contained directory per PR.
- Future versions support `repo-batch` format — grouping multiple PRs with
  shared blob stores.
- A `migrate` / `merge` command will combine multiple single-PR exports into
  batches.

## What Gets Captured

| Data            | Source         | Notes                                          |
| --------------- | -------------- | ---------------------------------------------- |
| PR metadata     | REST API       | Title, body, labels, branches, participants    |
| Review rounds   | REST + GraphQL | Formal reviews with state and body             |
| Review threads  | GraphQL        | Anchored conversations with resolution state   |
| Review comments | REST + GraphQL | Full comment bodies, markdown + HTML           |
| Code snapshots  | REST API       | Full file contents at each reviewed SHA        |
| Context files   | REST API       | Unchanged files in same package/directory      |
| Diffs           | REST API       | Base-to-snapshot and inter-round diffs         |
| Push timeline   | REST API       | Commits between rounds, force-push detection   |
| GitHub IDs      | All APIs       | Node IDs, numeric IDs for canonical references |
