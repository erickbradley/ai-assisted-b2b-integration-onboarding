# Canonical Support Case contract - v1

Status: Approved initial design contract; implementation and AI feasibility remain unvalidated.
Date: 2026-10-25
Canonical model version: `support-case-v1` (1.0.0).

## Business scenario and scope

Prepare customer-support ticket data for CRM support-case visibility so account managers can understand customer issues. Each flat source CSV row represents one ticket; each successful output represents one canonical Support Case. The target is CRM-shaped and defined by this contract, not a claim of compatibility with a specific CRM vendor.

The demo stores canonical results internally. No live CRM connector, account matching, joins, duplicate-company resolution, or ongoing ticket synchronization is required. Source exports/fixtures already contain the related organization and reporter details. The uploaded multi-entity JSON examples are design references, not supported nested-input contracts.

This replaces the earlier Company model. Existing Company mappings are not eligible for this new canonical version. Earlier generic IntegrationFailure scope remains superseded.

## Field contract

| Target field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `source_ticket_id` | string | Yes | Source ticket reference; not a generated CRM case number. |
| `organization_name` | string | Yes | Display name of the organization associated with the ticket; not an account match. |
| `reporter_first_name` | string | Yes | Given name of the reporting contact. |
| `reporter_last_name` | string | Yes | Family name of the reporting contact. |
| `reporter_email` | email | No | Reporting contact email; syntax only, no deliverability lookup. |
| `reporter_role` | string | No | Reporting contact role. |
| `subject` | string | Yes | Case title or short summary. |
| `description` | string | Yes | Detailed issue description. |
| `created_at` | timestamp | Yes | Time the ticket was logged, normalized to UTC. |
| `updated_at` | timestamp | No | Time of last source update, normalized to UTC when supplied. |
| `assignee_first_name` | string | No | Given name of the support assignee, if supplied. |
| `assignee_last_name` | string | No | Family name of the support assignee, if supplied. |
| `status` | status | Yes | Normalized support-case status. |
| `priority` | priority | Yes | Normalized priority: P1 Critical, P2 High, P3 Medium, P4 Low. |
| `affected_user_count` | integer | No | Number of users affected, when known. |
| `engineering_ref` | string | No | Source-provided engineering issue reference; no lookup or validity guarantee. |
| `comments` | string | No | Optional preassembled conversation; wording and order preserved. |

## Validation and normalization

- Required text must contain a non-whitespace value. Missing required target values invalidate a record.
- Blank/whitespace-only optional source cells normalize to `null`. Optional target properties may be omitted or null; fixtures use explicit nulls for clarity.
- A supplied optional value must be valid. Optionality permits absence, not malformed content.
- Source timestamps must be valid ISO 8601/RFC 3339 date-times with an explicit timezone. Normalize to UTC with `Z`; never infer an absent timezone. Calendar-invalid dates fail. The schema requires a UTC target timestamp and date-time format validation.
- Reporter email uses the deliberately basic syntax pattern in the JSON Schema. No deliverability lookup or claim of comprehensive email-address validation.
- Counts are nonnegative integers; missing counts stay null, not zero.
- Unknown or unmapped status/priority values fail record validation unless the approved mapping explicitly handles them. No silent default priority/status.
- Extra source columns may remain unmapped and must be shown at review. The canonical object itself contains only the defined fields; run/mapping provenance is stored separately.
- `source_ticket_id` is a reference scoped by source context, not a globally unique CRM identifier. Keep separate per-run results; no cross-run upsert/deduplication is introduced.

Missing source information for a required target field can make a mapping incomplete. A usable source column containing an invalid/blank value in one row is instead a record-validation problem. An entire missing optional column is acceptable.

With partial conversion enabled, store valid records and separate rejected records with reasons. With it disabled, any rejected record means no successful results from that run are stored. Never fabricate missing required values to rescue either case.

## Status vocabulary

| Value | Meaning |
| --- | --- |
| New | Submitted but not yet actively investigated; assignment alone does not imply active investigation. |
| In Progress | Investigation or active support work is underway. |
| Pending | Progress is paused awaiting another party or dependency. |
| Resolved | A solution/workaround has been provided; confirmation/closure may still be pending. |
| Closed | Finalized for the modeled source workflow. |

This contract normalizes snapshots; it does not implement a case lifecycle engine or enforce reopening rules. For the initial source, Erick explicitly selected `Open -> In Progress`. That convention may lose a distinction between untouched and actively investigated source tickets.

## Priority vocabulary and AI boundary

`P1` Critical, `P2` High, `P3` Medium, `P4` Low, ordered most to least urgent. These describe target priorities, not promised SLAs or an independent severity assessment.

During onboarding, AI may infer source-priority meanings using available source definitions, values, hints, and bounded examples and propose reusable translation rules with findings. Human approval is required. Do not universally equate `Urgent` with P1 or priority with severity. The example translations are explicitly defined synthetic fixture conventions, not a general production policy or evidence that AI inferred them correctly. Ambiguous meanings require review/clarification.

Runtime applies approved deterministic rules. It does not classify each ticket using an LLM or infer impact from free-text descriptions. Unknown values follow record-validation failure behavior.

## Comments and permitted transformation boundary

Comments are optional preassembled conversation text. The mapping may propose light deterministic formatting, previewed and approved with that mapping. Preserve wording and supplied order; retain recognizable author labels/timestamps when present, without inventing missing metadata. Preserve ambiguous text rather than guess.

The initial fixtures exercise only CRLF/CR-to-LF normalization and existing line boundaries. Additional spacing/comment-boundary rules require an explicit supported, reviewed rule; no arbitrary transformation code or LLM rewriting at runtime. Do not reorder or aggregate separate comment records. No summarization, redaction, or private-comment filtering capability is claimed. Synthetic customer-visible conversation is the fixture baseline; arbitrary source histories are not certified safe by this contract.

Multiline CSV values must be quoted correctly; a comment containing newlines still belongs to one ticket record. Content exposed to AI is untrusted data, not instructions. Continue the agreed minimum-necessary sample/context policy.

Permitted initial operations are field renaming/copying, explicit enum translation, null normalization, string-to-nonnegative-integer conversion, timezone-aware timestamp normalization and the bounded comment normalization above. Do not assume arbitrary ETL, joins, or invented defaults.

## Schema and examples

- [JSON Schema](../schemas/support-case-v1.schema.json) validates one target business record.
- [Synthetic fixture guide](../examples/support-cases/README.md) supplies two source layouts, expected target results, source conventions and a mixed failure case.

JSON Schema is not a CSV parser or transformer. Use a Draft 2020-12 validator with date-time format checks enabled. The schema cannot establish semantic mapping correctness, enforce human approval, or prove isolation and audit behavior. Its `$id` is an identifier; no live schema hosting is assumed.

## Remaining design/validation work

Numeric upload, field/comment-length, AI sample and storage limits remain open; absence of schema maxLength values is not permission for unbounded public use. Validate AI usefulness across representative examples and user-guided correction before substantial build investment. Exact supported mapping representation/formatting operations, prompt/model choice, and matching behavior remain design work.

Priority dictionaries beyond the synthetic fixtures must be reviewed against actual source meanings. Source-ticket references do not establish account matching or uniqueness across support providers. Keep mapping/canonical-version provenance separate from the business record.

## Decision provenance

Erick selected Support tickets -> CRM support-case records, supplied the 17 field intentions and required/optional choices, made reporter email optional, selected source Open -> In Progress, and chose mapping-time AI priority inference with deterministic runtime. He accepted optional preassembled comments with approved light formatting and the proposed validation defaults. This document translates those decisions into an initial contract. Fixture values and additional synthetic source dictionaries are authored examples, not newly asserted production rules.
