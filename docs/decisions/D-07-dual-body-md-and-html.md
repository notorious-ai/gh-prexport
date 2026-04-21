---
status: accepted
date: 2026-04-21
---

# D-07: Both markdown body and HTML body

## Context and Problem Statement

Every comment, review, and PR description in GitHub has two canonical
representations: the raw markdown the author typed, and the HTML GitHub renders
server-side. Those are not the same text. GitHub's renderer understands
extensions that plain CommonMark does not — suggestion blocks, formatted tables,
autolinked references, emoji shortcodes, `@`-mentions resolved to profile links
— so the HTML is often the only faithful record of what the reviewer actually
saw. Conversely, the raw markdown is what training pipelines and silver-tier
transformations expect to consume. The archive has to pick which to preserve on
every body-bearing record.

## Considered Options

- **Raw markdown only.** Store `body`; leave rendering to consumers.
- **HTML only.** Store `body_html`; discard the original markdown in favour of
  the rendered form.
- **Store both.** Keep `body` and `body_html` on every record that has a body.

## Decision Outcome

Chosen option: _store both_, because neither representation subsumes the other.
GitHub-rendered HTML carries information that cannot be reconstructed from the
markdown without replicating GitHub's renderer (with its evolving extensions and
repo-specific context); raw markdown carries authorial intent that cannot be
reconstructed from HTML without heuristic de-rendering. Storing both lets
consumers choose based on what they are doing — markdown for training-example
construction, HTML for faithful replay and UI reconstruction — without forcing
either to synthesise the other.

## Consequences

- Every body-carrying record in the schema — PR description, review summary,
  thread comment, review comment — exposes two fields: `body` for markdown,
  `body_html` for rendered HTML. This shape is visible in the concrete payloads
  in [reference/03][ref-03].
- Archive size grows modestly per comment. The overhead is a roughly constant
  factor over storing one form; text bodies are small compared to file contents,
  so the cost sits far below the blob store in the storage budget.
- Visualisation consumers (e.g., the playground named in the mission) can
  reproduce what the reviewer saw pixel-for-pixel without calling GitHub. That
  is part of what makes the archive [self-contained][sc].
- Training consumers that want only markdown can ignore `body_html` entirely.
  The decision does not impose a format on them; it just ensures the HTML is
  there when needed.
- GitHub's renderer output is GitHub-version-specific. The HTML captured at
  export time is a snapshot of that rendering; if a consumer needs rendering
  from a different GitHub vintage, a silver-tier transform can re-render the
  markdown with an alternative pipeline.

## More Information

- [Reference/03 — schema specification][ref-03] for the `body` / `body_html`
  field layout across record types
- [Mission success criterion 4][sc] — self-containment is what motivates keeping
  the HTML alongside the markdown

[ref-03]: ../reference/03-schema-specification.md
[sc]: ../mission.md#success-criteria
