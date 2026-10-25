# Architecture design drivers

> Scope update (2026-10-25): v1 now uses flat support-ticket CSV uploads and a CRM-shaped canonical Support Case model. See [Support Case contract](support-case-contract-v1.md) for fields, validation and examples. This supersedes Company-centered wording; live CRM delivery, joins and ongoing synchronization remain outside v1.

Status: Step 1 discussion summary; design inputs, not a selected architecture or validated implementation.
Date: 2026-10-25

## Purpose

Identify the requirements and constraints that most strongly shape the first architecture draft. This is a concise guide to design attention, not a replacement for the functional or nonfunctional requirements. All mandatory requirements still apply.

## Primary drivers

| Driver | Architectural consequence |
| --- | --- |
| Useful AI-assisted mapping | Evaluate whether proposals and user-guided revisions produce correct, complete, unambiguous mappings, and recognize missing information without fabricating source values. Prompt and model selection require representative tests; inference success alone does not demonstrate mapping correctness. |
| Personally affordable operation | Use a provisional deployed-demo operating ceiling of **$20/month**, excluding existing subscriptions. Keep idle costs low, measure consumption, and enforce limits on cost-generating activity. A feasible implementation within this ceiling has not been established. |
| Human-controlled, deterministic processing | Separate proposal/review from actual execution. Require explicit human confirmation of the exact reviewed mapping version before a processing run; execute that confirmed mapping deterministically. |

### Human approval boundary

- Intake and validation must occur before review. A bounded transformed sample is permitted for preview.
- Preview must not create a processing run, store successful business results, or make an unapproved mapping reusable.
- Both AI-generated and reused mappings require confirmation before actual processing.
- Approval must identify the exact reviewed mapping version. If a revision produces version 2, confirmation of version 2 authorizes that version, not an unspecified latest mapping.
- Approve and Revise are mutually exclusive workflow choices. Backend enforcement must preserve this guarantee even for stale views or delayed requests; the implementation mechanism remains open.
- Incomplete or ambiguous mappings remain ineligible for approval and execution.

## Mandatory supporting constraints

Carry these into the design even though the three drivers above receive particular attention:

- Isolate private workspace files, results, rejected records, and run data while allowing approved mapping definitions to be shared without private content.
- Preserve exact mapping/canonical-model version provenance and append-only audit metadata.
- Apply the defined business-data retention and early-eviction rules separately from retained audit metadata and mapping-definition retention.
- Enforce partial-conversion behavior, including no successful-result storage when partial conversion is off and any record fails.
- Prevent accidental duplicate processing requests; retain immutable original run outcomes and link user-initiated recovery runs.
- Respect the small interactive workload: CSV required, initially 50 data records per file, and a target of three concurrent visitors.
- Preserve the existing AI attempt limits, data-minimization rules, human-verification requirement, and failure/timeout behavior.

## Open questions and validation

1. Use the initial [Support Case contract](support-case-contract-v1.md) and [synthetic examples](../examples/support-cases/README.md) to evaluate mapping correctness; these are design artifacts, not measured AI results.
2. Test useful AI proposals and user-guided revisions before substantial build investment. Neither a model nor a prompt is selected by this summary.
3. Determine mapping representation and compatibility evaluation, including whether the reuse path can work during AI unavailability.
4. Prepare the first operating-cost estimate as soon as a provisional service design is available (Step 6), before implementation commitments. Include idle costs, expected use, development/test consumption where applicable, and a bounded higher-use scenario. Refine using measured inference costs.
5. Determine usage quotas, storage cap, and financial shutdown threshold. The **$20/month ceiling is not the shutdown trigger**: the trigger needs headroom for ongoing charges, reporting delay, and work already admitted. Its value and mechanism remain unresolved.
6. Retain existing unresolved response/timeout, retention, and recovery parameters for later design and validation rather than inventing values here.

## Decision provenance

Erick identified AI inference for mapping, personal budget, and transformation/storage as the initial drivers. Discussion sharpened the third into human-controlled deterministic processing; storage, isolation, audit, and retention remain required design concerns. Erick explicitly confirmed approval must bind to the reviewed mapping version and selected the provisional $20/month ceiling, requesting the earliest viable cost estimate.

This summary records those choices and reviewer observations. It does not claim AI feasibility, budget feasibility, or service selection has been validated.

The $20/month ceiling is a provisional constraint recorded here and reflected in the NFR quantitative table. Detailed quotas and financial shutdown values remain open.

## Source documents

- [Problem definition](problem-definition.md)
- [Functional requirements](functional-requirements-v1.md)
- [Nonfunctional requirements](nonfunctional-requirements-v1.md)
- [Pre-build application profile](application-profile-pre-build.md)

Next step: establish the system context and boundary before decomposing internal components or selecting AWS services.
