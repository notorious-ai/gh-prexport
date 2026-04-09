# Systems Engineering Documentation

This directory contains the systems engineering artifacts for gh-prexport,
following INCOSE's vision-to-product paradigm. Each addressable item has its own
file; cross-references use bracket notation with relative links.

## Structure

| Directory       | Contents                                              | ID Scheme |
| --------------- | ----------------------------------------------------- | --------- |
| `reference/`    | Original design documents (read-only archive)         | —         |
| `conops/`       | Concept of operations: actors, environment, workflows | W-nn      |
| `needs/`        | Functional needs bridging ConOps to requirements      | N-nn      |
| `requirements/` | Traceable requirements with V&V methods               | R-xxx-nn  |
| `analysis/`     | Analysis documents resolving design questions         | A-nn      |
| `decisions/`    | Numbered design decisions with rationale              | D-nn      |

## ID Schemes

- **W-nn** — Workflow (e.g., W-01 Single-PR Export)
- **N-nn** — Functional need (e.g., N-01 Review Round Reconstruction)
- **R-xxx-nn** — Requirement, where `xxx` is the domain:
  - `EXP` — Export pipeline
  - `SCH` — Schema and data format
  - `API` — GitHub API interaction
  - `CLI` — CLI and user experience
  - `STO` — Storage and archive structure
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
