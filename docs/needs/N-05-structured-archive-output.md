# N-05: Structured Archive Output

## Statement

The system shall emit the export as a structured archive: a predictable layout
of named, machine-readable resources in which the entities named by the other
needs — rounds, threads, snapshots, diffs, file contents — are each reachable by
a stable path and carry a stable shape.

## Rationale

Every consumer of the archive (downstream pipelines, fine-tuning jobs,
evaluation harnesses, a future replay playground) is a program, not a person. A
program needs three things the other needs do not guarantee on their own:

- a **predictable layout**, so that reaching a resource does not require
  guessing while scanning the archive;
- **stable shapes**, so that a parser written today still works on archives
  produced tomorrow (the export is the contract, not the code that wrote it);
- **discoverability**, so that the full set of rounds, threads, snapshots, and
  diffs in an archive can be enumerated without out-of-band knowledge.

Without this need the other needs are satisfied only in principle: the data is
captured, but a consumer has no reliable way to reach it.

This need says that structure exists and is stable; it does not say what the
structure is. The concrete filenames, directory layout, and field names belong
to the [R-SCH][r-sch] and [R-STO][r-sto] requirements that follow from this
need, informed by the existing [reference/02][ref02] and [reference/03][ref03]
design documents.

Traces to [W-01 §Archive assembly][w01-archive] and to [mission SC #1][sc] (the
output conforms to a schema specification, and all referenced files exist in the
archive).

## Acceptance Sketch

The need is met when:

- The archive is a single self-described unit: opening it, a program can
  determine the export's schema version and locate every resource the other
  needs require, without consulting any out-of-band index.
- Each kind of entity — PR metadata, rounds, threads, snapshots, diffs, file
  contents — has one canonical location and one canonical shape within the
  archive. Two archives produced by the same tool version are layout-identical
  up to the data they carry.
- The archive is read with general-purpose tooling (file-system traversal, JSON
  parser, patch reader). No archive-specific binary format or custom parser is
  required.
- A consumer can enumerate all rounds of a PR, all threads on a PR, and all
  snapshots of a round by reading the archive alone — no scan of unrelated
  files, no inference from filename patterns beyond what the layout documents.

[w01-archive]: ../conops/W-01-single-pr-export.md#archive-assembly
[sc]: ../mission.md#success-criteria
[r-sch]: ../requirements/
[r-sto]: ../requirements/
[ref02]: ../reference/02-directory-structure.md
[ref03]: ../reference/03-schema-specification.md
