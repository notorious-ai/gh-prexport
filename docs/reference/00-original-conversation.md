# Original Design Conversation

The design documents in this directory ([01][ref-01] through [05][ref-05]) were
produced in a single Claude Chat session on 3-4 April 2026. The full
conversation with thinking blocks is preserved as the canonical source in
[00-design-conversation.json](00-design-conversation.json) (22 messages, 11
turns, 5 artifacts).

This document distills the conversation arc, surfaces the key design-shaping
moments, and notes how each influenced the systems engineering artifacts now
being built on top of the reference docs.

## Conversation Structure

The JSON contains `chat_messages` (22 entries) and `artifacts` (5 files). Each
message has a `content` array of typed blocks (`text`, `thinking`).

```sh
# Count turns and total text
jq '.chat_messages | length' docs/reference/00-design-conversation.json
# → 22 (11 human, 11 assistant)

# List all human messages (the user's voice)
jq -r '.chat_messages[] | select(.sender=="human") | .content[] | select(.type=="text") | .text' \
  docs/reference/00-design-conversation.json

# Extract a specific thinking block (e.g., turn 3 where rounds were conceived)
jq -r '.chat_messages[5].content[] | select(.type=="thinking") | .thinking' \
  docs/reference/00-design-conversation.json
```

## The Arc: 11 Turns in 13 Hours

### Turn 1: Is there an existing tool?

> Is there an open source tool that scrapes my GitHub account for all my PR
> comments and reviews and exports it so that I can fine tune LLM agents on it
> later?

**Answer**: No. The closest match was a per-PR Chrome extension. Claude proposed
a ~200-line Go CLI with `gh` as the auth layer, emitting JSONL of
`(diff_hunk, comment)` pairs. This initial framing was flat and context-free;
the conversation would reshape it entirely.

### Turn 2: What data matters?

> What would be the most comprehensive list of information that is useful for
> later use with LLMs to replicate and learn the user's code practices?

Claude decomposed PR interactions into 8 surfaces: code context envelope, role
signal, PR-level metadata, conversation thread structure, review verdict,
commit-level evolution, temporal/social context, and the "negative space" (what
the reviewer did not comment on). The proposed `PRInteraction` record was
comprehensive but temporally flat.

### Turn 3: The pivotal correction, rounds

> This conceptual scheme doesn't reflect the relevant points in time. My reviews
> often involves several "rounds". Each time I'd go on the GitHub UI, place a
> review with many comments, sometimes even comment without a review (by mistake
> mostly), and then submit without approval.

**This is the single most consequential input in the conversation.** The user
introduced the "round" concept directly from their own review workflow. Claude
recognized it immediately (thinking: "The user is pointing out that my schema
was too flat — it didn't capture the temporal/round-based nature of code
reviews") and restructured the entire data model around rounds as the
fundamental unit.

The user also described the **"giving in" pattern**: strong objection →
developer pushback → softened position → acceptance with conditions. This
pattern motivated the thread conversation model's preservation of the full arc
across rounds, and later influenced [N-09][n09] (target user perspective) and
the thread-round association model.

### Turn 4: Scope decisions

> Pure API for now, this is a GitHub CLI extension, developed with Go. The focus
> of this project is on exporting data, no analysis yet. Data duplication is
> fine with me at this point. As a reference signal, consider we'd create a
> playground to explore and visualise past PRs, based on the exported data
> alone.

Four scope-setting decisions in one message: technology (Go, `gh`, pure API),
philosophy (export not analysis), economics (storage is cheap), and the
**self-containment test** (the playground). The playground became a formal
success criterion in [mission.md](../mission.md) and motivated the
[Visualization Consumer](../conops/actors.md#visualization-consumer) actor.

### Turn 5: Thread realism

> I still think you put too much into threads, people crap all over them. Some
> people just resolve and forget to fix, others wait for the reviewer to
> maintain the conversation, ignoring it on GitHub altogether.

The user pushed back on Claude's thread resolution inference. Claude's thinking
block acknowledged: "I'm over-engineering the thread structure by trying to
infer semantics from data that's inherently noisy and incomplete." This directly
produced design decision D-02 (no inference at export time) in
[reference/05][ref-05]: capture only what GitHub's API returns (`is_resolved`,
`is_outdated`), leave interpretation to the analysis layer.

### Turn 6: Thread-round association

> It's ok to keep placeholders for some insights, like that resolution type.
> Also, round is importantly but I think per thread rather than per comment.

A targeted structural correction: the round association belongs on the
**thread**, not on individual comments. A thread is born in a round; the
conversation under it is timeless. This became design decision
[D-01 (threads belong to rounds, comments don't)][d-01].

### Turn 7-8: Format and the medallion model

> I agree with Directory tree with content-addressable blob store. This is the
> "bronze tier" data (borrowed from the Medallion model of OLAP).

Claude had proposed three format options (single JSON, directory tree, SQLite).
The user chose the directory tree with content-addressed blobs and positioned
the project within the medallion model: bronze (raw export) → silver (structured
training examples) → gold (agent quality metrics).

The user also established the **per-PR isolation** strategy: start with one
independent directory per PR, add a merge command later. This became D-06
(single-PR export as atomic unit) and shaped
[W-02](../conops/W-02-batch-and-merge.md).

### Turn 9: File splitting

> I wonder if there's value in keeping all threads as a single file, and all
> rounds too. Consider both merging into existing/new files and splitting into
> others.

Claude's thinking block analyzed the scaling characteristics: "Rounds: there are
typically few rounds per PR (2-10 usually). The data per round is small.
Threads: A PR could have 2 threads or 200 threads." This led to the current
split: rounds stay in `pr.json` (always small), threads get their own
`threads.json` (scales with discussion volume), and file contents are
externalized to the blob store.

### Turn 10: Final corrections

> Just to clarify on threads: comments don't logically "span" rounds. A thread
> usually belongs to a specific review round [...] and the conversation belongs
> to that thread — timeless. Also, the threads file should not be keyed by file,
> rather have an index that shows different groupings.

Two final refinements: (1) thread conversations are timeless, not cross-round;
(2) `threads.json` uses a flat array with precomputed indexes (`by_file`,
`by_round`, `by_resolution`) rather than a file-keyed structure. The user also
requested canonical GitHub IDs throughout (node IDs, numeric IDs).

### Turn 11: Artifact generation

> Generate artifacts in this Claude project so that we can reference this plan
> more easily.

Claude produced the five reference documents that now live in this directory.

## Influence Map

How the conversation shaped the systems engineering artifacts:

| Conversation moment               | SE artifact influenced                                                                                 |
| --------------------------------- | ------------------------------------------------------------------------------------------------------ |
| "No existing tool" (turn 1)       | [mission.md](../mission.md) — problem statement                                                        |
| 8 data surfaces (turn 2)          | [W-01](../conops/W-01-single-pr-export.md) — export pipeline steps                                     |
| Round concept (turn 3)            | [glossary](../glossary.md) — "round" and "round construction" terms                                    |
| "Giving in" pattern (turn 3)      | [N-09][n09] — target user perspective                                                                  |
| Playground test (turn 4)          | [mission.md](../mission.md) — success criteria; [actors](../conops/actors.md) — visualization consumer |
| Export not analysis (turn 4)      | [mission.md](../mission.md) — scope boundaries                                                         |
| Thread realism (turn 5)           | D-02 — no inference at export time                                                                     |
| Thread-round association (turn 6) | [D-01][d-01] — threads belong to rounds                                                                |
| Medallion model (turn 8)          | [mission.md](../mission.md) — data architecture position                                               |
| Per-PR isolation (turn 8)         | D-06 — single-PR atomic unit; [W-02](../conops/W-02-batch-and-merge.md)                                |
| File splitting rationale (turn 9) | [reference/02][ref-02] — directory structure                                                           |
| Precomputed indexes (turn 10)     | D-08 — indexes in threads.json                                                                         |

[ref-01]: 01-design-overview.md
[ref-02]: 02-directory-structure.md
[ref-05]: 05-glossary-and-decisions.md
[n09]: ../needs/N-09-target-user-perspective.md
[d-01]: ../decisions/D-01-threads-belong-to-rounds.md
