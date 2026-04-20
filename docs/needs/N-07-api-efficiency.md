# N-07: API Efficiency

## Statement

The system shall export a pull request within a bounded and predictable budget
of GitHub API calls that scales with the pull request's shape (rounds, threads,
unique files) rather than with naïve request counts, so that the archives the
other needs describe remain attainable in practice and not only in principle.

## Rationale

The GitHub API is a shared, rate-limited resource. The user's `gh` token carries
a single hourly budget (5 000 REST requests, 5 000 GraphQL points per
[environment §Network Context][env-net]); every request the export issues is one
the user cannot issue from anywhere else that hour. A tool that exhausts the
budget on a single PR is not a tool the user will run twice.

Three shapes of waste produce this outcome and this need exists to rule them
out:

- **Redundant file fetches across rounds.** Most files do not change between
  rounds of a review. Fetching the same file five times because it appears in
  five snapshots turns a 50-file PR into 250 requests for no additional
  information. Content-addressed dedup by blob SHA is the natural remedy; the
  need constrains the outcome (no redundant fetch), not the mechanism.
- **Wrong API surface.** Threads fetched one REST comment at a time and stitched
  via `in_reply_to_id` chains cost dozens of requests; the same data arrives in
  one or two GraphQL queries ([reference/04 §Rate Limiting][ref04-rate]).
  Choosing the efficient surface is not a performance optimisation — it is what
  keeps the export within budget.
- **No progress on retry.** When an export is interrupted mid-run (rate limit,
  network blip), a naïve re-run starts over. Any data already captured in the
  archive — especially blobs, which dominate the request count — should be
  skipped on a subsequent run for the same PR.

A supplementary mechanism is worth flagging here, even if this need does not
mandate it: blob content need not arrive through the REST or GraphQL surface at
all. A `git clone` or `git fetch` of the repository delivers every blob the
export needs under a separate, much more permissive rate regime. Treating git as
an alternative blob source — when the target repository is clonable under the
authenticated user's credentials — collapses the dominant term in the API budget
and is a legitimate avenue for satisfying this need. The choice of when and how
to fall back to git is a later design question; the need leaves room for it.

Traces to [environment §Network Context][env-net],
[W-01 §Rate limit exhaustion][w01-rate], and
[reference/04 §Rate Limiting Considerations][ref04-rate].

## Acceptance Sketch

The need is met when:

- The number of API calls issued for a single-PR export scales with the PR's
  shape (rounds × unique files + threads, roughly), not with `rounds × files` in
  the worst case. A file whose content has not changed between two snapshots is
  fetched once, not twice.
- For a typical PR (a few rounds, tens of threads, tens of changed files,
  bounded context scope) the export consumes a small fraction of the hourly API
  budget — leaving headroom for other tools the user runs against the same
  token.
- Re-running the export for a PR whose previous run was interrupted does not
  re-fetch blobs that were already written to the archive. The tool makes
  progress toward completion on each run; it does not restart.
- When GitHub exposes both REST and GraphQL surfaces for the same data, the
  exporter uses the surface whose cost scales better with the volume of data —
  not the one that happens to be simpler to call.

[env-net]: ../conops/environment.md#network-context
[w01-rate]: ../conops/W-01-single-pr-export.md#rate-limit-exhaustion
[ref04-rate]: ../reference/04-api-mapping.md#rate-limiting-considerations
