# Glossary

Shared terminology for gh-prexport systems engineering artifacts. Each term
includes the document where it is first defined or most fully described.

## Domain Terms

| Term                   | Definition                                                                                                                                               | Origin                            |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **Target user**        | The GitHub user whose review practices are being exported. All data is captured from their perspective.                                                  | [reference/01][ref-01] §Purpose   |
| **Round**              | A single review session: the target user opens the PR at a specific commit, places comments, and optionally submits a formal review with a verdict.      | [reference/01][ref-01] §Rounds    |
| **Thread**             | A conversation anchored to a code location (file + line), initiated in a specific round. Contains chronologically ordered comments from any participant. | [reference/01][ref-01] §Threads   |
| **Snapshot**           | The state of all PR-affected files (and context files) at a specific commit SHA.                                                                         | [reference/01][ref-01] §Snapshots |
| **Blob**               | A file's content, stored by its git blob SHA for deduplication.                                                                                          | [reference/01][ref-01] §Blobs     |
| **Context file**       | A file not changed in the PR but stored alongside changed files to provide surrounding context (e.g., other files in the same Go package).               | [reference/05][ref-05] §Glossary  |
| **Gap**                | The period between two consecutive rounds, including developer pushes, reply comments, and elapsed time.                                                 | [reference/05][ref-05] §Glossary  |
| **Loose comment**      | A review comment submitted by the target user without an associated formal review (e.g., submitting a single comment instead of batching into a review). | [reference/05][ref-05] §Glossary  |
| **Round construction** | The algorithm that assembles rounds from raw GitHub API data: reviews sorted by `submitted_at`, loose comment grouping, and gap computation.             | [reference/04][ref-04] §Rounds    |

## Data Architecture Terms

| Term                          | Definition                                                                                                               | Origin                             |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| **Bronze tier**               | Raw exported data with no analysis, inference, or transformation. First stage of the Medallion model.                    | [reference/01][ref-01] §Purpose    |
| **Medallion model**           | A data architecture pattern organizing data into bronze (raw), silver (cleaned/structured), and gold (aggregated) tiers. | [reference/01][ref-01] §Purpose    |
| **Export archive**            | The self-contained directory tree produced by a single export run, containing all JSON, patch, and blob files.           | [reference/02][ref-02] §Layout     |
| **Content-addressed storage** | File storage keyed by git blob SHA, enabling natural deduplication when the same file appears across multiple snapshots. | [reference/02][ref-02] §Blobs      |
| **Target user perspective**   | The organizing principle of the export: rounds, threads, and gaps are defined relative to the target user's activity.    | [reference/05][ref-05] §Decision 1 |

[ref-01]: reference/01-design-overview.md
[ref-02]: reference/02-directory-structure.md
[ref-04]: reference/04-api-mapping.md
[ref-05]: reference/05-glossary-and-decisions.md
