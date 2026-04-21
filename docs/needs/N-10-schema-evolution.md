# N-10: Schema Evolution

## Statement

The system shall produce archives that identify their own schema version and
format, so that a consumer can detect — before parsing any content — which shape
the archive carries and refuse or adapt as appropriate. The schema shall be
allowed to change across versions without ambiguity about which version a given
archive belongs to.

## Rationale

The archive format defined today is not the archive format forever. The design
already foresees at least two transitions: the move from a single-PR layout to
the repo-batch format ([W-02][w02]), and refinements to individual files' shapes
as gap markers, context scoping, and other questions from Phases 4–6 get
resolved. Both will change what consumers see, and neither is avoidable.

Bronze-tier discipline permits breaking schema changes — consumers that break
should update to the new format — but it does not permit silent ones. The
minimum a consumer needs from an archive is the ability to answer "which version
of the schema is this?" before trusting any other field. Without that, a parser
written for v1 will quietly misread a v2 archive, and the training data,
evaluation data, or replay built on top will be wrong in ways nothing in the
export will flag.

This need therefore mandates a self-declared schema version, not a particular
evolution policy. Semantic versioning, date-stamped versions, or format-family
labels like `single-pr` vs `repo-batch` are all permissible mechanisms; the need
constrains the outcome (detectable, unambiguous, declared up front), not the
convention. The specific labels and the transition policy belong to the
[R-ARCHV][r-archv] requirements and the migrate/merge command that [W-02b][w02b]
introduces.

Traces to [reference/01 §Versioning & Evolution][ref01-versioning],
[W-02b §Merge into Repo-Batch Format][w02b], and to [mission SC #1][sc] (the
output conforms to a schema specification — which specification must be knowable
from the archive alone).

## Acceptance Sketch

The need is met when:

- Every archive carries a schema version identifier at a well-known,
  trivially-locatable position (a top-level file or root-level field) that a
  consumer can read without parsing the rest of the archive.
- The identifier distinguishes both the format family (e.g., single-PR versus
  repo-batch) and the schema version within that family, so that consumers can
  decide whether they understand the archive before reading it.
- Two archives produced by the same tool version carry the same schema
  identifier; archives produced by different tool versions that happen to emit
  compatible content may share an identifier, but not by accident — version
  equality is an explicit claim of compatibility, not a coincidence of output.
- The version identifier is stable across the archive's lifetime: a consumer who
  reads the identifier, caches it, and later re-reads the archive gets the same
  answer. If a future tool rewrites an existing archive in place, it rewrites
  the identifier too.
- A consumer encountering an unknown schema version can refuse cleanly — the
  identifier is sufficient to detect the mismatch — rather than producing
  silent, partial, or wrong results.

[w02]: ../conops/W-02-batch-and-merge.md
[w02b]: ../conops/W-02-batch-and-merge.md#w-02b-merge-into-repo-batch-format
[ref01-versioning]: ../reference/01-design-overview.md#versioning--evolution
[sc]: ../mission.md#success-criteria
[r-archv]: ../requirements/
