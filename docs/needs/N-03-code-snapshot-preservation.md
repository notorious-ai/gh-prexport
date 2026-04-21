# N-03: Code Snapshot Preservation

## Statement

The system shall preserve the state of the code the target user was looking at
during each round: for every round's snapshot SHA, the archive shall record the
set of files that made up the reviewed surface and retain the content of those
files, so that a downstream consumer can read any commented line in its
surrounding file at the exact version the reviewer saw.

## Rationale

A comment anchored to `foo.go:142` only teaches what "good review" looks like
when a consumer can read `foo.go` as it stood when the comment was placed.
Without the snapshot, a thread is a disembodied remark; with it, the thread
becomes an example of judgment exercised against real code.

Snapshots also make the archive survive change in the outside world. GitHub will
age out force-pushed commits, repositories get renamed, access tokens expire.
The commitment that the archive be replayable with no GitHub API calls
([mission SC #4][sc]) rests almost entirely on snapshots: everything else in the
export — threads, rounds, diffs — is metadata; the file contents are the
substrate those metadata point at.

Two properties follow from the bronze-tier philosophy:

- File contents are content-addressed by git blob SHA so that an unchanged file
  across several rounds is stored once. Data duplication is acceptable at this
  tier, but it should fall out of the data model, not be manufactured by the
  exporter.
- The reviewed surface includes both files changed in the PR and surrounding
  context files that were unchanged. Context is what makes a comment
  interpretable; how "surrounding" is scoped is an open analysis item
  ([A-02][a-02]) and is not fixed by this need.

Traces to [W-01 §Snapshot assembly][w01-snapshot] and
[§Blob fetching][w01-blob], and to [mission SC #2][sc] (downstream consumer can
reconstruct the full review timeline with complete file context at each round)
and [mission SC #4][sc] (archive is self-contained).

## Acceptance Sketch

The need is met when, for every round recorded in the archive:

- The archive carries a manifest for that round's snapshot SHA, listing every
  file that makes up the reviewed surface with its path, change status, and a
  reference to the content that file held at that SHA.
- A consumer reading the archive offline can, given a thread's file and line,
  retrieve the file's contents at the snapshot SHA of the round the thread was
  born in, without any network call.
- A file whose content has not changed between two snapshots in the same PR is
  stored once and referenced from both manifests.
- Files the exporter cannot or will not fetch in full (binary, oversized,
  unreachable after force-push) are recorded in the manifest with an explicit
  status that distinguishes them from files whose content is available — never
  silently omitted.

[sc]: ../mission.md#success-criteria
[w01-snapshot]: ../conops/W-01-single-pr-export.md#snapshot-assembly
[w01-blob]: ../conops/W-01-single-pr-export.md#blob-fetching
[a-02]: ../analysis/
