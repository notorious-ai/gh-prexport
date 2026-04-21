# N-11: Archive Integrity Check

## Statement

The system shall provide a way to verify, against a single archive on disk, that
every reference the archive makes to data inside itself resolves — and that
every explicit gap marker is what the archive is permitted to carry in place of
that data. The verification shall be executable without contacting GitHub.

## Rationale

[N-06][n06] names the property that every reference inside the archive resolves
to data inside the archive. It does not provide the reader with a way to find
out whether the property actually holds on a given archive. That affordance is
what this need adds.

The two needs are complementary:

- **N-06 is the property.** It tells the exporter what to produce.
- **N-11 is the verifier.** It tells the consumer how to check what was produced
  before trusting it.

A verifier is worth stating as its own need for two reasons. First, a consumer
who did not run the exporter — a data scientist handed an archive by someone
else, an evaluation harness reading archives from a shared store, a future
playground loading an archive from a URL — has no standing to assume N-06 was
satisfied; they need a way to check. Second, the verifier is also the test
mechanism for the [N-08][n08] `--strict` contract: an archive produced with
`--strict` should contain no gap markers, and an independent verifier is how a
consumer confirms that claim without re-running the export.

The verification is local to one archive. Cross-archive consistency (for
example, checking that two archives produced for the same PR agree on round
metadata) is a separate question that belongs to the future repo-batch workflows
([W-02][w02]), not to this need.

Traces to [N-06][n06] (the property being checked), [N-08][n08] (the
graceful/strict contract the verifier reports against), and to the `check`
subcommand described in the project memory that motivated this need.

## Acceptance Sketch

The need is met when:

- The system exposes a verification capability that takes a single archive as
  input and returns a pass/fail verdict plus a list of the conditions that
  produced the verdict.
- A passing archive has no reference that points at missing data, and every gap
  marker it contains is one the schema defines as permitted in place of that
  data.
- A failing archive's report names each broken reference or disallowed gap
  marker by its location in the archive, so a reader can find and investigate it
  without guessing.
- The verifier reads only the archive. It does not fetch anything from GitHub,
  the original repository, or any external service to reach its verdict, so it
  works on an archive whose source has since become unreachable.
- An archive produced with `--strict` (per [N-08][n08]) passes the verifier only
  if it contains zero gap markers; an archive produced in the default graceful
  mode may pass the verifier while carrying gap markers, provided each one is
  schema-permitted.
- The verifier's pass verdict is a sufficient precondition for every downstream
  consumer that assumes [N-06][n06]. Consumers do not have to re-implement
  reference resolution to gain confidence in the archive.

[n06]: N-06-self-contained-replay.md
[n08]: N-08-graceful-degradation.md
[w02]: ../conops/W-02-batch-and-merge.md
