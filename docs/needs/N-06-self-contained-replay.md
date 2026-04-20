# N-06: Self-Contained Replay

## Statement

The system shall produce archives that are self-contained: given only the files
in the archive, a consumer shall be able to replay the target user's review of
the pull request — the rounds, the threads, the code each comment was placed on,
the diffs between rounds — without consulting GitHub, the original repository,
or any other live service.

## Rationale

The closure property is the single acceptance test for the whole tool. It is the
criterion the original design fixed as the **playground test**: an offline
visualization that can render any past PR entirely from the exported data, with
no API calls at render time ([reference/01 §Philosophy][ref01-philosophy]).
Every other need in this directory describes one resource the archive must
carry; this need describes the property the archive gains only when those
resources are all present together.

Self-containment is also what makes the archive outlive the world it was
exported from. Repositories get renamed and deleted, access tokens expire,
force-pushes age commits out, private repos go read-only. An archive that
requires live GitHub access to interpret is not a bronze-tier dataset; it is a
cache. Self-containment converts the cache into a permanent record.

Two properties follow:

- Every reference the archive contains (a commit SHA a thread is anchored to, a
  blob SHA a snapshot manifest points at, an author login attached to a comment)
  is either resolvable inside the archive or explicitly marked as unresolved.
  There are no silent dangling pointers that only a GitHub call could follow.
- Interpreting the archive requires no credentials. A consumer with the archive
  on a laptop offline has access to everything a consumer with network access
  would have.

Traces to [mission SC #4][sc] (archive is self-contained: no GitHub API calls
are needed to interpret any part of it) and to
[W-01 §Archive assembly][w01-archive].

## Acceptance Sketch

The need is met when:

- Every round, thread, snapshot, diff, and file reference in the archive
  resolves to data present in the archive itself. Anything that cannot be
  resolved is carried as an explicit gap marker (per the graceful-default
  posture N-01..N-04 establish), not as a bare reference that a reader would
  have to resolve out-of-band.
- A consumer can render the full review timeline — who said what, on which code,
  in what order, with the surrounding file contents — from the archive alone,
  with no network access and no credentials.
- The archive contains no URL, API endpoint, or external identifier that a
  reader _must_ dereference to make sense of any captured datum. External
  identifiers may appear as provenance (e.g., a GitHub node ID kept for
  traceability), but the archive's meaning does not depend on resolving them.

[ref01-philosophy]: ../reference/01-design-overview.md#philosophy
[sc]: ../mission.md#success-criteria
[w01-archive]: ../conops/W-01-single-pr-export.md#archive-assembly
