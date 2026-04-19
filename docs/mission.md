# Mission Analysis

## Problem Statement

Code review is one of the highest-leverage activities in software engineering: a
senior reviewer's feedback encodes architectural judgment, quality standards,
and contextual awareness that takes years to develop. Yet this expertise is
trapped in GitHub's pull request UI, scattered across thousands of threads,
buried under newer activity, and inaccessible to any system that could learn
from it.

There is no structured, machine-readable dataset of code review interactions.
Without one, LLM agents lack the high-quality training and validation material
needed to replicate a team's review practices. The feedback that a senior
reviewer gives on a connection pool cleanup, a test boundary, or a naming choice
exists only as ephemeral UI state on GitHub; it cannot be studied, searched, or
used to teach an agent what "good review" looks like for a specific team.

## Mission Statement

Capture a team's code review practices as structured, self-contained archives
that serve as both training data and ground-truth validation material for LLM
coding agents.

## Scope

### In scope

- Export of pull request review interactions: reviews, threads, comments, code
  snapshots, diffs, and push timelines
- Structuring at export time: round construction, thread-to-round mapping, gap
  computation, and precomputed indexes
- Self-contained archives that can be read without GitHub API access
- Content-addressed deduplication of file contents within an export
- Single-PR export as the atomic unit, with a path toward batch merging

### Out of scope

- Analysis, inference, or scoring of review quality
- Classification of comment intent (suggestion, question, nit, blocking)
- Resolution inference (whether a thread was addressed, deferred, or ignored)
- Transformation of data into training examples (that is the downstream
  pipeline's responsibility)
- Visualization or replay tooling (those are separate consumers of the archive)

## Success Criteria

A v1 export of a single PR is successful when:

1. The output conforms to the [schema specification][ref-03] and all referenced
   files (blobs, patches, snapshots) exist in the archive
2. A downstream consumer can reconstruct the full review timeline: who said
   what, on which code, in what order, with complete file context at each round
3. Round construction accurately reflects the actual review sessions as they
   happened, with faithful thread-to-round attribution and gap computation
4. The archive is self-contained: no GitHub API calls are needed to interpret
   any part of it

## Stakeholders

| Stakeholder        | Role                   | Interest                                                                                                                                       |
| ------------------ | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Reviewer**       | Data source            | The senior reviewer whose interactions are captured. Begins with a single user; the vision is team-wide adoption.                              |
| **Data scientist** | Data consumer          | Transforms the bronze-tier export into fine-tuning examples and validation sets for LLM coding agents. May be the same person as the reviewer. |
| **LLM agent**      | Downstream beneficiary | The system being trained or evaluated against the exported review data. Not a direct user of gh-prexport.                                      |

## Position in the Data Architecture

gh-prexport produces **bronze-tier** data in the [Medallion model][medallion]:
raw, unprocessed, faithful to what GitHub knows. The same export serves two
downstream purposes:

- **Fine-tuning**: training examples derived from the review interactions teach
  an LLM agent to replicate the team's review style
- **Evaluation**: the exported ground truth (what the human reviewer actually
  said, on what code, in what context) becomes the validation set for measuring
  whether the agent's review output matches human judgment

The bronze-to-silver transformation (structuring exports into training examples)
and silver-to-gold aggregation (measuring agent quality across many PRs) are
separate systems that consume the archive gh-prexport produces.

[ref-03]: reference/03-schema-specification.md
[medallion]: glossary.md#data-architecture-terms
