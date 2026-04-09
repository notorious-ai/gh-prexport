# Actors

The system's actors fall into two categories: those present during export
(operational actors) and those who consume the archive afterward (downstream
actors). The export's design serves both, but only operational actors interact
with gh-prexport directly.

## Operational Actors

### Reviewer (target user)

The senior reviewer whose code review interactions are being captured. This is
the person running the export and the perspective from which all data is
organized: rounds are their review sessions, threads are anchored to their
comments, and gaps measure the intervals between their activity.

In the initial version, the reviewer is also the person invoking the CLI. The
vision is team-wide adoption where multiple reviewers each export their own
review history.

**Motivation**: preserve the reasoning, standards, and patterns embedded in
their review feedback before it becomes inaccessible.

### GitHub Platform

The authoritative source of all review data. GitHub provides the REST and
GraphQL APIs that gh-prexport reads from, and its data model (reviews, review
threads, review comments, timeline events) defines the boundaries of what can be
captured.

GitHub is both a data source and a constraint: API rate limits (5,000 REST
requests/hour, 5,000 GraphQL points/hour) bound export throughput, and data
availability depends on repository access permissions and the permanence of
historical SHAs.

### gh CLI

The runtime environment for gh-prexport. The `gh` CLI provides authentication
(OAuth token management), extension distribution (`gh extension install`), and
API access (`gh api`). gh-prexport relies on `gh` for all GitHub communication;
it does not manage credentials or tokens directly.

### PR Author (passive)

The developer who submitted the pull request and responds to review feedback.
The author's commits, force-pushes, and reply comments are captured as part of
the review timeline, but the author does not interact with gh-prexport. Their
activity constitutes the "response signal" that completes each review round:
what the developer changed (or did not change) in response to the reviewer's
feedback.

## Downstream Actors

### Data Scientist

The person who transforms the bronze-tier export into fine-tuning examples and
validation sets for LLM coding agents. May be the same person as the reviewer.

The data scientist needs the export to be self-contained (no API calls needed to
interpret it), structurally consistent across PRs (for batch processing), and
rich enough to reconstruct the full review context including code state,
conversation arcs, and the reviewer's acceptance threshold.

### Visualization Consumer

A hypothetical downstream system (e.g., a playground) that replays past PRs
entirely from the exported data. This actor validates the self-containment
guarantee: if the archive is sufficient to render a full interactive PR replay
without any GitHub API access, the export is complete enough for any downstream
purpose.
