---
status: accepted
date: 2026-04-21
---

# D-01: Threads belong to rounds, comments don't

## Context and Problem Statement

A review round is a single session in which the target user opens the PR at a
specific commit, comments, and optionally submits a verdict. A thread is a
conversation anchored to a code location, and the comments inside it can arrive
from any participant at any time — the PR author's reply after the next push,
the reviewer's follow-up two rounds later, and so on. The design must decide
where the round association lives: on the individual comments, or on the thread
as a whole.

The question surfaced during the 3–4 April 2026 design conversation at
[Turn 6][turn-6]: the initial draft attached a round identifier to every
comment; the target user pushed back, observing that "round is important but I
think per thread rather than per comment."

## Considered Options

- **Per-comment round association.** Every comment records the round in which it
  was written. Round membership follows chronology exactly.
- **Per-thread round association.** The thread records the round it was born in;
  comments under it carry no round of their own.

## Decision Outcome

Chosen option: _per-thread round association_, because a thread is meaningful
only relative to the round that opened it. Subsequent messages — a developer's
reply, the reviewer's follow-up in a later round — are continuations of that
conversation, not fresh participants in other rounds. Tagging each comment with
a round misattributes replies to rounds they were never part of and fragments
the thread's identity as a single arc.

The conversation under a thread is therefore treated as timeless: comments
appear in chronological order with authorship and timestamps, and the thread
carries the round-of-origin as a first-class field.

## Consequences

- Threads become stable containers for discussion: a reader reconstructs "who
  said what, on which code, in what order" without inferring round boundaries
  comment-by-comment, which is exactly what [mission success criterion 2][sc]
  asks for.
- Developer replies and cross-round follow-ups no longer inflate later rounds
  with content that did not originate there, keeping round-level reviewer
  behaviour interpretable for downstream analysis.
- Cross-round evolution of a thread (a reviewer softening position two rounds
  later, for example) is recoverable only from comment timestamps and the commit
  SHAs on individual comments, not from a round field on the comment. Tooling
  that wants per-round slices of a thread computes them from those signals.
- The thread's round-of-origin is null when the initiator is not the target
  user, because the concept of a round is target-user-centric (see [N-02][n02]).

## More Information

- [Original design conversation, Turn 6 — Thread-round association][turn-6]
- [N-02: Thread Capture][n02] — primary downstream consumer of this decision
- [Mission success criterion 2][sc] — review timeline reconstruction

[turn-6]: ../reference/00-original-conversation.md#turn-6-thread-round-association
[n02]: ../needs/N-02-thread-capture.md
[sc]: ../mission.md#success-criteria
