# N-08: Graceful Degradation

## Statement

The system shall produce a useful archive from partial data by default,
recording every condition that prevented a referenced object from being captured
as an explicit, machine-readable gap marker rather than as a silent omission or
an export failure. A consumer opting into strict completeness (`--strict`) shall
receive the opposite guarantee: the export succeeds only when no gap marker
would be written, and fails otherwise.

## Rationale

Every other need in this directory describes what the archive carries when the
world cooperates. This need describes what happens when the world does not. Real
pull requests routinely produce partial data: force-pushes age out snapshot
SHAs, the API size cap excludes large files, binary files are not reasonable to
store inline, rate limits interrupt mid-run. A policy that refuses to produce
any archive unless everything was retrieved would fail on the majority of real
PRs; a policy that silently omits what could not be retrieved would corrupt the
archive's meaning. Neither is acceptable, and neither is what this need calls
for.

### Why graceful is the default

Graceful archives preserve strictly more information than strict ones. Whatever
a strict run would emit, a graceful run emits as well — plus a labelled list of
the gaps. A consumer who wants strictness can filter out any archive that
carries a gap marker; a consumer who wants coverage cannot recover data that was
never written. The default therefore serves the richer superset. It also aligns
with the bronze-tier principle of capturing what GitHub knows, honestly and in
full — including the fact that some things could not be known.

The primary consumer named in [mission.md][mission-stakeholders], the data
scientist training an LLM agent, benefits from coverage. Partial archives are
still training signal; dropped archives are not.

### What `--strict` is for

Evaluation and ground-truth consumers have the opposite need. A validation set
whose archives may quietly contain absent snapshots is not a reliable benchmark.
For these consumers the `--strict` flag converts every gap condition into an
export failure, so that any archive the tool produces is complete by
construction. A strict archive needs no gap-aware reader; a graceful archive
does.

### What counts as a gap

This need inherits gap conditions from the other needs rather than enumerating
them exhaustively. In scope are, at minimum:

- an unreachable snapshot SHA or any of its derived data ([N-03][n03],
  [N-04][n04]);
- a file whose content the exporter cannot or will not fetch in full (binary,
  oversized, unreachable) ([N-03][n03]);
- any condition subsequent needs and requirements introduce that would cause an
  explicit "absent" marker to be written.

The exhaustive list is established as the other needs and requirements land;
this need supplies the dual-mode contract that wraps them all.

Traces to [W-01 §Error Scenarios][w01-errors], to [mission SC #1][sc] (the
output conforms to schema — gap markers must be schema-known), and to
[mission SC #4][sc] (a graceful archive with explicit gap markers is still
self-contained in the sense [N-06][n06] requires).

## Acceptance Sketch

The need is met when:

- In the default mode, no failure to retrieve a referenced object causes the
  export to abort. Every such failure is recorded in the archive as a typed,
  machine-readable marker that a consumer can enumerate without heuristics.
- Explicit gap markers are part of the archive's schema (not ad-hoc strings, not
  omissions): a consumer written against the schema can distinguish a gap from a
  present value without inference.
- A graceful archive remains self-contained in the [N-06][n06] sense — gap
  markers resolve to "this was missing, here is what is known about it" inside
  the archive, not to a prompt for an API call.
- `--strict` causes the export to fail — with a clear report of which conditions
  triggered the failure — whenever any gap marker would have been written in the
  default mode. An archive produced with `--strict` contains no gap markers.
- The two modes produce the same archive whenever there are no gaps to record.
  Strictness is a contract, not a different output shape.

[mission-stakeholders]: ../mission.md#stakeholders
[sc]: ../mission.md#success-criteria
[w01-errors]: ../conops/W-01-single-pr-export.md#error-scenarios
[n03]: N-03-code-snapshot-preservation.md
[n04]: N-04-diff-generation.md
[n06]: N-06-self-contained-replay.md
