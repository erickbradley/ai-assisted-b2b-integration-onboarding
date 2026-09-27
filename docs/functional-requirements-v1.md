# AI-Assisted B2B Integration Onboarding Platform

## v1 Functional Requirements

Status: Final\
Date: 2026-09-13\
Updated: 2026-09-27 — AI limits, approval eligibility, retention, provenance, preview, and audit-data boundaries aligned\
Milestone: Trello Card #6 — Define functional requirements

## 1. Purpose and current scope

The PoC demonstrates onboarding company-data files into a canonical Company model. It combines reuse of previously approved mappings, AI-assisted mapping when reuse is not possible, human confirmation, deterministic transformation, audit reporting, and bounded recovery of rejected records.

This scope supersedes the earlier runtime `IntegrationFailure` normalization and simulated remediation/replay direction.

## 2. Actors

- **Demo user:** uploads data, reviews mapping proposals and transformation previews, confirms or rejects mappings, requests AI revisions, views results, and exports rejected records.
- **Integration service provider:** owns approved mapping definitions, establishes demo limits, and provides the canonical Company model.

The PoC uses a temporary session-level demo identity and logically isolated workspace without account registration. For v1, “current user” means the visitor's temporary session identity. That identity scopes private workspace data, usage limits, and current-user mapping-search precedence during the session. Returning visitors do not need to resume the same identity, workspace, or prior work. Enterprise registration, identity administration, roles, and approval hierarchies are outside v1.

## 3. File intake and validation

**FR-001 — Manual upload.** The first interaction must allow the user to upload a company-data file for processing.

**FR-002 — Supported formats.** CSV is required. JSON is a non-blocking v1 stretch goal. YAML is deferred.

**FR-003 — Format validation.** The system must reject an unsupported or malformed file and present a useful error.

**FR-004 — Data-presence validation.** The system must reject an empty file or a file without data records and present a useful error.

**FR-005 — Record limit.** The demo must enforce an operator-configured maximum number of data records per file, initially set to 50. A CSV header does not count toward the limit. The system must tell the user when a file exceeds the current limit and reject the entire file before mapping search or AI inference begins. It must not truncate the upload or process only the records within the limit.

## 4. Approved-mapping discovery

**FR-006 — Search before AI.** The system must search approved mappings before requesting an AI-generated mapping.

**FR-007 — Search order.** It must search mappings approved under the current temporary session identity newest-to-oldest, followed by service-owned shared mappings approved in other sessions newest-to-oldest. It must select the first compatible mapping found. The session-level identity provides search precedence and private provenance; it does not make the mapping definition user-owned or prevent approved mappings from becoming reusable across sessions.

**FR-008 — Compatibility outcome.** Exact structural identity is not required. A mapping is compatible only when it can support a complete and unambiguous conversion to the current canonical Company model.

**FR-009 — Missing information.** A mapping is incompatible when information required by the canonical model is absent. The system must not fabricate missing source values.

**FR-010 — Additional source fields.** Extra source fields do not make an otherwise compatible mapping ineligible. They may remain unmapped.

## 5. AI-assisted mapping

**FR-011 — AI fallback.** If no approved mapping is compatible, the system must ask AI to propose a mapping to the canonical Company model.

**FR-012 — Unsuccessful proposal.** When AI cannot produce a complete and unambiguous mapping, the system must present its findings or failure reasons. It must not invent missing source data.

**FR-013 — Revision hints.** A user may request an AI revision by supplying written guidance about what should change. Direct editing of mapping rules is outside the v1 interaction.

**FR-014 — Retry limits.** The v1 demo must allow one initial AI mapping attempt and no more than two additional AI revision attempts within the same mapping workflow. The system must show remaining attempts and prevent additional inference after the limit is reached. The limit may be revisited after real inference costs are measured, as described in NFR-5.4; it is not an unresolved initial value.

**FR-015 — Retry exhaustion.** At the retry limit, the user may confirm the latest proposal only if it is complete and unambiguous, or discard the upload. If the proposal remains incomplete or ambiguous, confirmation and execution must be blocked; the system must show the unresolved findings and allow the user to discard the upload. Reaching the retry limit does not relax approval eligibility or permit further AI attempts.

## 6. Human review and confirmation

**FR-016 — Review information.** Before confirmation and a processing run, the system must show:

- Source-to-canonical mapping rules
- A preview of transformed sample records
- Every source field that will remain unmapped
- The mapping version and canonical Company model version

This requirement applies to reused and AI-generated mappings. Unused source fields do not invalidate an otherwise complete mapping.

**FR-017 — Human control.** A mapping is eligible for approval and execution only when it can satisfy every required field in the canonical Company model completely and unambiguously, without fabricating missing source values. This applies to both reused and AI-generated mappings; unused source fields do not invalidate an otherwise complete mapping. Before confirmation, the system may apply the proposed mapping only to a bounded sample for the preview required by FR-016. Preview evaluation must not create a processing run, store successful business results, or make the mapping reusable. No mapping may be used for an actual processing run until the user confirms it.

**FR-018 — Review actions.** The user may confirm the mapping, reject and discard it, or request an AI revision with hints while attempts remain.

## 7. Mapping ownership and versioning

**FR-019 — Service ownership and reuse.** Every approved mapping definition is owned by the integration service provider and automatically becomes reusable across users.

**FR-020 — No proprietary content in shared definitions.** A reusable mapping definition must not contain source records, sample values, user prompts, user identity, or private audit history.

**FR-021 — Immediate reuse.** Each approved mapping version becomes immediately eligible for future searches.

**FR-022 — Version search.** Compatible versions are searched newest-to-oldest. Older versions remain searchable when a newer version does not match and remain retrievable during their defined mapping-retention period. After that period, an obsolete mapping definition may be deleted while historical audit retains its mapping ID/version and provenance metadata. This is a demo/PoC concession under NFR-7.6; production may require archival or retention for as long as historical runs depend on the definition. The mapping-retention duration remains open and is separate from the 24-hour business-record review window.

**FR-023 — No ordinary activation state.** Normal mapping versioning does not require a separate active/inactive designation.

**FR-024 — Canonical-model binding.** Every mapping must record the canonical Company model version against which it was approved.

**FR-025 — Canonical-model change.** When the canonical Company model version changes, mappings from earlier model versions become ineligible until revalidated. Earlier mapping definitions remain subject to the retention policy in FR-022 and NFR-7.6; a canonical-model change does not itself authorize immediate deletion.

Canonical-model administration and automated mapping revalidation are outside v1.

## 8. Transformation behavior

**FR-026 — Partial-conversion control.** Before processing begins, the user must be able to set `Allow partial conversion`. It defaults to Yes/On.

**FR-027 — Partial conversion enabled.** Valid records must be transformed and stored in the demo's canonical data store, which represents successful downstream delivery and serves as the theoretical system of record for v1. Rejected records must be separated and assigned failure reasons. v1 does not require delivery to a separate target application.

**FR-028 — Partial conversion disabled.** If any record fails, the entire file run must fail and no transformed results from that run may be stored in the demo's canonical data store. The failure must still be audited.

**FR-029 — Results.** The user must be able to view the transformed results and the rejected records produced by a run.

## 9. Audit reporting and run status

**FR-030 — File-level audit.** Each run must record what was processed, when it ran, who initiated it, the result, the exact mapping ID/version used, the canonical-model version, and accepted/rejected record counts. During the referenced mapping definition's retention period, its rules must be retrievable so a reviewer can determine how that run mapped the data. A duplicate copy of the rules in each run is not required. After an obsolete definition is deleted under FR-022/NFR-7.6, the audit reference remains but rule inspection is no longer guaranteed; this limitation is a demo/PoC concession.

**FR-031 — Record-level review details.** During the applicable review/retention window, each record outcome must show its before-conversion fields, whether it was accepted or rejected, and the rejection reason when applicable. These business-record values are temporary review data, remain private to the user's workspace, and must be deleted under the applicable retention and storage-pressure policies. They are not part of the permanent append-only audit history.

The permanent audit/provenance record retains metadata such as identifiers, event types, timestamps, actors, status, counts, failure classifications, mapping and canonical-model version references, and reprocessing relationships. It must not retain expired source, transformed, or rejected-record values.

**FR-032 — Run statuses.** Each run receives one of these immutable statuses:

- **Full Success/Complete:** every record was transformed and stored.
- **Partial Success/Complete:** some records were transformed and stored, while unresolved rejected records remain.
- **Total Failure:** no transformed records were stored. This includes whole-file rejection when partial conversion is off.

**FR-033 — Immutable run outcome.** Later recovery must not change the original run's status.

**FR-034 — Linked runs.** Reprocessing creates a new run linked to the original and receives its own audit record and status.

## 10. Rejected-record recovery

**FR-035 — Retention.** Rejected records must be retained until successfully resolved/reprocessed or the 24-hour review window expires, whichever comes first. The 24-hour limit is specifically a demo/PoC concession, not a production retention requirement. Records also remain subject to the separate demo storage-pressure policy in NFR-7.7–7.8, which permits earlier removal with a visible notice while retaining historical audit metadata. The numeric total-storage ceiling remains open.

**FR-036 — Failure classification.** Before recovery, the system must classify a rejection as either potentially mapping-correctable or an unrecoverable source-data-quality failure.

**FR-037 — Direct export.** Missing, invalid, or otherwise unrecoverable source-data failures must bypass mapping search and AI and proceed to export.

**FR-038 — Alternate mapping.** For a mapping-correctable rejection, the system must first search approved mappings for another compatible candidate.

**FR-039 — AI reevaluation.** If no approved mapping resolves a mapping-correctable rejection, the user may request AI reevaluation within the configured usage limits.

**FR-040 — Recovery review.** Any alternate or AI-revised mapping must return to human review and confirmation before it is applied.

**FR-041 — Unresolved export.** If recovery is unsuccessful or declined, the user must be able to export the original rejected records in the same format in which they were uploaded.

## 11. Demo experience

**FR-042 — Downloadable sample.** The upload screen must provide a downloadable, editable synthetic CSV. It must not duplicate this with a separate in-app sample runner.

**FR-043 — JSON sample.** A downloadable JSON sample is included only if JSON support ships.

**FR-044 — Synthetic-data restriction and acknowledgment.** Before upload, the demo must state that it is intended only for synthetic data and is not intended or designed to handle proprietary, private, sensitive, regulated, or otherwise protected information. Visitors must acknowledge that restriction before uploading. The demo must also explain the supported format and record limit. The acknowledgment records the visitor's assertion; it does not verify the contents or transfer responsibility for enforcing the platform's safeguards.

**FR-044A — Suspected-sensitive-data safeguard.** Where the platform can reasonably detect potentially private or sensitive data, it must pause further AI-assisted processing, explain the concern, and allow the current demo user to continue only after explicitly asserting that the detection is a false positive and the submitted data contains no prohibited private or sensitive information. The override does not authorize processing data that actually violates FR-044, and the safeguard must not be represented as guaranteeing detection.

**FR-045 — Guided proof.** The sample and demo state must make it possible to demonstrate both paths: AI proposal when no mapping matches and approved-mapping reuse on a later compatible upload.

**FR-046 — Exploration.** Visitors may edit the downloaded sample before uploading it to explore compatibility, unmapped-field notices, transformation failures, partial conversion, recovery, and export.

**FR-047 — Human verification and public fallback.** Any person who discovers the public link may view the project explanation and architecture information. Before starting an interactive demo session or cost-generating processing, the visitor must complete a brief human-verification step that does not require account registration. Successful verification creates the temporary session-level identity and workspace described in Section 2. Failed verification must block interactive processing while leaving the public, non-cost-generating project explanation and architecture information accessible. Human verification is a deterrent and must not be represented as guaranteeing that subsequent activity is non-automated; usage and cost controls still apply.

A running-conversion visualization is optional and must not block v1 completion.

## 12. Functional acceptance scenarios

Card #6 is functionally satisfied when the following behaviors can be demonstrated or tested:

1. A valid CSV passes intake validation; malformed, empty, and oversized files produce useful errors. An oversized file is rejected in full before mapping search or AI inference begins; no records are selected for partial processing or silent truncation.
2. A compatible current-user mapping is found before shared mappings or AI are considered.
3. When no current-user mapping matches, a compatible shared mapping can be found without exposing another user's data.
4. When no approved mapping matches, AI proposes a mapping or explains why it cannot.
5. Reused and AI-generated mappings show rules, transformed samples, and unmapped fields before confirmation.
6. Transformation cannot begin without human confirmation.
7. A complete, unambiguous AI revision with user hints can be approved as a new reusable version. Incomplete or ambiguous mappings cannot be approved or executed, including after the retry limit is reached.
8. A mapping associated with an older canonical-model version is not reused.
9. With partial conversion on, valid records are stored and rejected records are separated with reasons.
10. With partial conversion off, one failed record produces Total Failure and stores no transformed results.
11. A mapping-correctable rejection follows alternate-mapping search, bounded AI reevaluation, and human confirmation.
12. An unrecoverable source-data failure bypasses AI and can be exported in the original input format.
13. Reprocessing produces a linked run with its own status while preserving the original run's status.
14. Retry, record, retention, and usage limits are visible and enforced.
15. Before upload, the visitor acknowledges the synthetic-data-only restriction. If the safeguard flags suspected private or sensitive data, AI-assisted processing pauses and continues only after the visitor makes the permitted false-positive assertion; the override does not authorize prohibited data.
16. A visitor who passes human verification receives a temporary session identity/workspace and may use the interactive demo. A visitor who fails verification cannot start interactive processing but can still view the public project explanation and architecture information.

## 13. Deferred capabilities

- YAML support
- JSON support if the stretch goal does not ship
- Scheduled, API/SDK, and event-triggered ingestion
- Enterprise identity, roles, and approval administration
- Canonical-model administration
- Automated or compatibility-aware mapping revalidation after canonical-model changes
- Production-scale storage, retention, and recovery controls
- Optional conversion-progress visualization
- The superseded runtime `IntegrationFailure` normalization and simulated remediation/replay workflow
