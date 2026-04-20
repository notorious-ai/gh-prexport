# N-09: Target User Perspective

## Statement

The system shall frame the archive around a single named **target user** and
organise every captured artifact relative to that user's activity: rounds are
the target user's review sessions, gaps are the intervals between them, and the
negotiation arc within each thread is captured as the target user's judgment
meeting the author's response.

## Rationale

A pull request is a multi-party conversation, but the archive gh-prexport
produces is not a neutral transcript of it. It is a record of _one reviewer's
craft_ — the standards they enforced, the feedback they gave, the positions they
softened on and the ones they held. Flattening that into "all review activity on
the PR" erases the thing the archive exists to preserve.

This perspective is the reason the data model is shaped the way it is, as set
out in the original design session ([reference/00 Turn 3][conv-turn3]):

> My reviews often involve several "rounds". Each time I'd go on the GitHub UI,
> place a review with many comments, sometimes even comment without a review (by
> mistake mostly), and then submit without approval.

"Rounds" as the fundamental unit ([N-01][n01]) only makes sense once there is a
privileged participant whose sessions define them; the same activity by a
co-reviewer would be other rounds in their own archive, not a merged stream in
this one. The **"giving in" pattern** the same turn described — strong objection
→ author pushback → softened position → acceptance with conditions — is likewise
a property of one reviewer's arc across rounds, not a per-comment fact. The
archive can only reconstruct it if it knows whose arc to follow.

Per-user archives are also what makes later merging (the future [W-02][w02]
workflow) meaningful: a repo-batch export is several target users' archives
combined, each still interpretable in its own right, not a de-identified blend.

Traces to [reference/01 §Purpose][ref01-purpose], the
[glossary][glossary-domain] entry on _target user perspective_, and
[reference/00 Turn 3][conv-turn3].

## Acceptance Sketch

The need is met when:

- The archive names exactly one target user as a first-class, schema-known
  entity, identifiable by a stable GitHub identifier (login plus node ID or
  numeric ID), not inferred from file names or directory paths.
- Rounds in the archive are the target user's review sessions only. Reviews,
  review comments, or loose comments by other participants are still captured
  where they belong to the archive's subject (for example as replies inside a
  thread the target user initiated), but they do not constitute rounds.
- Gaps between rounds are computed relative to the target user's `submitted_at`
  timestamps, not to any other participant's activity.
- Threads initiated by someone other than the target user have a null or
  explicitly "other" round origin (per [N-02][n02]); they are recorded, but they
  do not invent rounds that the target user did not conduct.
- Two archives produced for the same PR but with different target users are
  separately interpretable: each describes one reviewer's arc, and neither is a
  subset of the other in its choice of rounds, gaps, or thread origins.

## Future Evolution

The framing above — one target user, always in the reviewer role — is a
deliberate simplification. It matches the initial use case (a reviewer exports
their own reviews to fine-tune an agent on their own craft) and its simplicity
is a feature: it pins down what "round" means, who owns gaps, and whose arc the
archive follows, without branching on role.

As the tool sees more use and its archives meet more friction, this need is a
likely place to revisit. Two extensions are already foreseeable:

- **Author role.** A PR author's trajectory through a review — which feedback
  they pushed back on, which they conceded, how their commits evolved between
  rounds — is itself a training signal. A future version might allow the target
  user to be the author instead, with rounds and gaps redefined relative to
  pushes and replies rather than reviews.
- **Multiple reviewers.** A PR reviewed by two or more senior reviewers carries
  a multi-voice negotiation the per-user archive cannot capture. A future format
  could hold several rounds streams in one archive, either by combining per-user
  archives or by modelling round co-occurrence directly.

Neither extension is in scope for v1, and neither should inform the schema v1
decisions without concrete friction to motivate it. This section exists so that
future work has an obvious place to attach itself when it arrives, and so that
the current simplicity is not mistaken for an assertion that
one-reviewer-one-archive is the final shape.

[conv-turn3]: ../reference/00-original-conversation.md#turn-3-the-pivotal-correction-rounds
[ref01-purpose]: ../reference/01-design-overview.md#purpose
[glossary-domain]: ../glossary.md#domain-terms
[n01]: N-01-review-round-reconstruction.md
[n02]: N-02-thread-capture.md
[w02]: ../conops/W-02-batch-and-merge.md
