# Decision Records

Records in this directory use [MADR] format. Conventions specific to this
project are below.

## Frontmatter

```yaml
---
status: accepted
date: 2026-04-21
---
```

- `status` — one of `proposed`, `accepted`, `deprecated`, `superseded`,
  `rejected`.
- `date` — ISO-8601 `YYYY-MM-DD`; the date of the current status, not the date
  the record was created.
- `superseded-by` — required when and only when `status: superseded`. Its value
  is the `D-nn` ID of the replacement. Supersessors cross-link back in their own
  `## More Information`; the superseded record keeps its body so the history
  stays readable.

## Filename and ID

- Kebab-case with a `D-nn` prefix, e.g., `D-01-threads-belong-to-rounds.md`.
- `D-nn` is a flat, zero-padded two-digit sequence. IDs are never reused, even
  for rejected or superseded records.
- This README is intentionally unnumbered so existing `D-01`..`D-10` citations
  in committed artifacts stay valid.

## Links

Ref-style links targeting the most specific anchor available, per the SE docs'
house rule.

[MADR]: https://adr.github.io/madr/
