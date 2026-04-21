# Systems Engineering Documentation for gh-prexport

## Context

We are adopting INCOSE's "vision to product" paradigm, as demonstrated by
Stewart Gebbie (sgebbie) in resystems-io/renotify. Stewart spent 3 days writing
24 documentation commits before any code. His process: human-authored ConOps
first, then AI-assisted analysis, formal requirements, numbered decisions, gap
analysis, and finally a phased implementation plan.

gh-prexport already has 5 architecturally mature design documents (untracked in
git) covering schemas, API mapping, directory structure, and 10 unnumbered
design decisions. These become reference material. The work ahead extracts and
formalizes their content into proper SE artifacts, filling gaps (no ConOps, no
formal requirements, no traceability).

**This session**: Phase 0 (commit existing docs) + Phase 1 (mission and ConOps).
Implementation planning is NOT in scope for any of these phases; it follows as a
separate capstone once all SE artifacts exist.

**File convention**: every addressable item gets its own file. Cross-references
use bracket notation with relative links.

---

## Phase 0: Preserve and Scaffold (3 commits)

### Step 0.1 — Commit reference documents as-is

- Move the 5 existing docs into `docs/reference/` and commit
- Remove the empty `docs/adr/` directory
- These are a read-only archive from this point; future changes go into SE
  artifacts
- `docs: preserve existing design documents as reference material`

### Step 0.2 — Create docs scaffold and README

- Create `docs/README.md` explaining the document structure and ID schemes
  (N-nn, R-xxx-nn, A-nn, D-nn, W-nn)
- Create directories: `conops/`, `needs/`, `requirements/`, `analysis/`,
  `decisions/`
- `docs: scaffold systems engineering document structure`

### Step 0.3 — Create glossary

- `docs/glossary.md` with the 8 terms from reference doc 05, plus SE framework
  terms (round construction, bronze tier, medallion model, target user
  perspective)
- `docs: establish glossary with domain and systems engineering terminology`

---

## Phase 1: Mission and Concept of Operations (5+ commits)

Human-authored core. The human writes the mission and ConOps narrative; AI
assists with structure and consistency checking against reference docs.

### Step 1.1 — Mission analysis

- `docs/mission.md`
- Problem statement, mission statement, scope boundaries, success criteria,
  stakeholders, relationship to medallion model
- Draws from: reference/01 (Purpose, Philosophy), human insight
- `docs: define mission analysis for gh-prexport`

### Step 1.2 — Actors

- `docs/conops/actors.md`
- Reviewer (target user), Data Scientist (future consumers), GitHub Platform, gh
  CLI, PR Author (passive)
- Draws from: reference/01, reference/04, human input
- `docs: identify actors in the operational concept`

### Step 1.3 — Operational environment

- `docs/conops/environment.md`
- Execution context, network context (API rates), data context (repo states),
  storage context, constraints, assumptions
- Draws from: reference/04 (Rate Limiting), reference/02 (storage), human input
- `docs: describe the operational environment and constraints`

### Step 1.4 — Workflow: Single-PR export

- `docs/conops/W-01-single-pr-export.md`
- Narrative walkthrough of the primary use case, error scenarios, resumability
  considerations
- Draws from: reference/01, reference/04 (round construction), human input
- `docs: describe single-PR export workflow (W-01)`

### Step 1.5 — Workflow: Batch export and future merge

- `docs/conops/W-02-batch-and-merge.md`
- W-02a multi-PR loop, W-02b migrate --merge, W-02c incremental re-export
- These are post-MVP but shape the v1 archive format
- Draws from: reference/02 (repo-batch format), reference/01 (versioning)
- `docs: describe batch export and merge workflows (W-02)`
- CRITICAL: commit each workflow separately

---

## Phase 2: Functional Needs (2+ commits)

Needs bridge ConOps → requirements. High-level capability statements, not
implementation-specific. Each need gets its own file with ID, statement,
rationale (traced to workflow), and acceptance sketch.

### Step 2.1 — Core functional needs (N-01 through N-06)

- 6 files in `docs/needs/`
- N-01 Review Round Reconstruction, N-02 Thread Capture, N-03 Code Snapshot
  Preservation, N-04 Diff Generation, N-05 Structured Archive Output, N-06
  Self-Contained Replay
- `docs: define core functional needs N-01 through N-06`
- CRITICAL: commit each need separately

### Step 2.2 — Supporting functional needs (N-07 through N-11)

- 5 files in `docs/needs/`
- N-07 API Efficiency, N-08 Graceful Degradation, N-09 Target User Perspective,
  N-10 Schema Evolution, N-11 Archive Integrity Check
- N-11 was re-scoped during drafting. The originally planned framing ("archive
  must be internally consistent: blob references resolve, snapshot manifests
  exist for all round SHAs, diff files are present") turned out to duplicate
  N-06, which already mandates that every reference in the archive resolves to
  data present in the archive. N-11 is therefore the _verification_ complement
  to N-06: it names the capability of checking that the property N-06 asserts
  actually holds on a given archive. A future `check` CLI subcommand is the
  expected mechanism.
- CRITICAL: commit each need separately
- **Forward reference**: after creating N-09, update the `[n09]` link in
  `docs/reference/00-original-conversation.md` to point at the new file.
  Re-evaluate whether the influence map's characterisation still holds against
  the need as written.

---

## Phase 3: Requirements (5+ commits)

Requirements derived from needs + reference docs. Each file follows the
template: ID, statement, rationale (A1), trace to parent (A4), allocation (A8),
V&V method (A2).

Domain codes: R-EXP (export pipeline), R-SCH (schema/format), R-API (GitHub
API), R-CLI (CLI/UX), R-STO (storage/archive).

### Step 3.1 — Export pipeline requirements (R-EXP-01..08)

- Round construction, loose comment grouping, thread fetching, thread-to-round
  mapping, snapshot construction, context file discovery, diff generation,
  participant deduction
- `docs: define export pipeline requirements R-EXP-01 through R-EXP-08`

### Step 3.2 — Schema requirements (R-SCH-01..08)

- pr.json, threads.json, snapshot manifest, blob storage, diff storage,
  version.json, dual body (md+html), precomputed indexes
- `docs: define schema and data format requirements R-SCH-01 through R-SCH-08`

### Step 3.3 — API interaction requirements (R-API-01..06)

- GraphQL for threads, REST for metadata, blob dedup, rate limits, force-push
  detection, gh auth
- `docs: define GitHub API interaction requirements R-API-01 through R-API-06`

### Step 3.4 — CLI requirements (R-CLI-01..05)

- Invocation syntax, progress reporting, output directory, --target-user flag,
  exit codes
- `docs: define CLI and user experience requirements R-CLI-01 through R-CLI-05`

### Step 3.5 — Storage requirements (R-STO-01..04)

- Atomic self-contained directory, blob dedup within PR, directory layout,
  future repo-batch format
- `docs: define storage and archive requirements R-STO-01 through R-STO-04`

---

## Phase 4: Analysis (4 commits)

Resolve the 6 open implementation questions from reference/04-api-mapping.md.

### Step 4.1 — A-01: Loose comment grouping strategies

- Three approaches (timestamp window, commit SHA, unassigned), evaluation,
  recommendation → impacts R-EXP-02
- `docs: analyse loose comment grouping strategies (A-01)`
- **Forward reference**: after creating A-01, update the `[A-01]` link in
  `docs/conops/W-01-single-pr-export.md:36` to point at the new file.
  Re-evaluate whether W-01's description of the grouping problem still
  accurately frames the analysis.

### Step 4.2 — A-02: Context file scope and configurability

- Language-specific analysis (Go same-package, others same-directory), cost
  analysis, recommendation → impacts R-EXP-06, R-API-04
- `docs: define context file scope with language-aware defaults (A-02)`

### Step 4.3 — A-03: Force push handling

- Timeline API capabilities, what data is lost, graceful degradation strategy →
  impacts R-API-05, R-EXP-05, R-SCH-03
- `docs: resolve force push handling with graceful degradation (A-03)`

### Step 4.4 — A-04: Binary files, large files, PR conversation comments

- Combined analysis of the three smaller open questions → impacts R-SCH-01,
  R-SCH-04, R-API-03
- `docs: resolve binary files, large files, and PR comment scope (A-04)`

---

## Phase 5: Decisions (2 commits)

### Step 5.1 — Formalize existing decisions (D-01..10)

- 10 files from reference/05 decisions, each with: status, date, context,
  decision, alternatives, consequences, references
- `docs: formalize design decisions D-01 through D-10`

### Step 5.2 — New decisions from analysis (D-11..14)

- D-11 (loose comments by commit SHA), D-12 (context scope default), D-13
  (force-push graceful degradation), D-14 (PR conversation comments)
- `docs: record analysis-derived decisions D-11 through D-14`

---

## Phase 6: Gap Analysis and Traceability (3 commits)

### Step 6.1 — Gap analysis

- `docs/analysis/A-05-gap-analysis.md`
- Systematic comparison: reference docs vs. requirements coverage, schema fields
  without requirements, deferred items
- `docs: identify coverage gaps between reference docs and requirements (A-05)`

### Step 6.2 — Supplementary requirements

- New requirements to close gaps found in A-05 (estimated 3-5 files)
- `docs: add supplementary requirements to close coverage gaps`

### Step 6.3 — Traceability matrix

- `docs/traceability.md`
- Needs→Requirements, Requirements→Decisions, Requirements→Analysis,
  Requirements→Reference matrices with coverage summary
- `docs: create traceability matrix linking needs, requirements, decisions, and analysis`

---

## What comes after (NOT in scope for these phases)

Once Phase 6 is complete, the next step is the **implementation plan** — a
phased, checkboxed document that an agent can execute against. That will be
authored collaboratively in a separate session. By then, every requirement will
be traceable to a need, every design question resolved, and every decision
documented.

---

## Progress

**Phase 0**: Complete (Steps 0.1–0.3). Reference docs committed, scaffold
created, glossary established.

**Phase 1**: Complete (Steps 1.1–1.5). Mission, actors, environment, W-01, W-02
committed. Original design conversation preserved with influence map.

**Phase 2**: Complete. Steps 2.1 (N-01–N-06) and 2.2 (N-07–N-11) landed as
separate commits per-need. N-11 was re-scoped during drafting from "archive
integrity invariant" (which duplicated N-06) to "archive integrity check" (the
verification capability that confirms N-06's invariant holds on an archive), as
described in Step 2.2.

**Phase 3**: Next. Pull in the reference docs (`reference/03` for schema,
`reference/04` for API mapping, `reference/02` for storage layout) to derive the
R-xxx requirements. Commit each requirement separately.

Original Phase 2 sourcing note: the
[original design conversation][conversation.json] (distilled in
[00-original-conversation.md][conversation.md]) is the source for fine details
not in the reference docs.

[conversation.json]: docs/reference/00-design-conversation.json
[conversation.md]: docs/reference/00-original-conversation.md
