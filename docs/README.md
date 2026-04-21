# Systems Engineering Documentation

This directory contains the systems engineering artifacts for gh-prexport,
following INCOSE's vision-to-product paradigm. Each addressable item has its own
file; cross-references use bracket notation with relative links.

## Structure

| Directory       | Contents                                              | ID Scheme |
| --------------- | ----------------------------------------------------- | --------- |
| `reference/`    | Original design documents (read-only archive)         | —         |
| `conops/`       | Concept of operations: actors, environment, workflows | W-nn      |
| `needs/`        | Functional needs bridging ConOps to requirements      | N-nn        |
| `requirements/` | Traceable requirements with V&V methods               | R-nn[-STUB] |
| `analysis/`     | Analysis documents resolving design questions         | A-nn      |
| `decisions/`    | Numbered design decisions with rationale              | D-nn      |

## ID Schemes

- **W-nn** — Workflow (e.g., W-01 Single-PR Export)
- **N-nn** — Functional need (e.g., N-01 Review Round Reconstruction)
- **R-nn[-STUB]** — Requirement. `nn` is a flat, universal sequence shared
  across all requirements; areas do not have their own counters. An optional
  `STUB` of up to 5 characters may follow the number to name the area the
  requirement concerns, purely to aid scanning and group references — it is
  not an enumeration, and areas need not partition cleanly. Favor short
  vowel-dropped abbreviations that read as subject-matter nouns. Stubs
  currently in use:
  - `ARCHV` — archive-wide contracts: root layout, self-description,
    schema version identifier, evolution policy
  - `REVW`  — review rounds and their commits
  - `THRD`  — comment threads
  - `SNAP`  — HEAD snapshots and blob contents
  - `DIFF`  — diffs between snapshots
  - `API`   — GitHub API interaction: query patterns, rate limits, auth
  - `CLI`   — CLI surface: subcommands, flags, output, exit codes
  - `GAPS`  — completeness: strict mode, gap markers, abort contract
  - `RPLY`  — self-contained replay: no out-of-band lookups
  - `TRACE` — provenance: fields that let a consumer follow a datum to
    its source
- **A-nn** — Analysis document (e.g., A-01 Loose Comment Grouping)
- **D-nn** — Design decision (e.g., D-01 Threads Belong to Rounds)

## Reading Order

1. [Mission Analysis](mission.md) — why this system exists
2. [ConOps](conops/) — who uses it, in what context, doing what
3. [Needs](needs/) — what capabilities the system must provide
4. [Requirements](requirements/) — what the system must do, traceably
5. [Analysis](analysis/) — how open questions were resolved
6. [Decisions](decisions/) — what was chosen and why
7. [Glossary](glossary.md) — shared terminology
8. [Reference](reference/) — original design documents (archive)
