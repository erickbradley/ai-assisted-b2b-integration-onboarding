# AI-Assisted B2B Integration Onboarding Platform

## v1 Functional Requirements

Status: Final\
Date: 2026-09-13\
Milestone: Trello Card #6 — Define functional requirements

## 1. Purpose and current scope

The PoC demonstrates onboarding company-data files into a canonical Company model. It combines reuse of previously approved mappings, AI-assisted mapping when reuse is not possible, human confirmation, deterministic transformation, audit reporting, and bounded recovery of rejected records.

This scope supersedes the earlier runtime `IntegrationFailure` normalization and simulated remediation/replay direction.

## 2. Actors

- **Demo user:** uploads data, reviews mapping proposals and transformation previews, confirms or rejects mappings, requests AI revisions, views results, and exports rejected records.
- **Integration service provider:** owns approved mapping definitions, establishes demo limits, and provides the canonical Company model.

The PoC uses lightweight demo identity. Enterprise registration, identity administration, roles, and approval hierarchies are outside v1.

## 3. File intake and validation

**FR-001 — Manual upload.** The first interaction must allow the user to upload a company-data file for processing.

**FR-002 — Supported formats.** CSV is required. JSON is a non-blocking v1 stretch goal. YAML is deferred.

**FR-003 — Format validation.** The system must reject an unsupported or malformed file and present a useful error.

**FR-004 — Data-presence validation.** The system must reject an empty file or a file without data records and present a useful error.

**FR-005 — Record limit.** The demo must enforce an operator-configured maximum number of data records per file, initially set to 50. A CSV header does not count toward the limit. The system must tell the user when a file exceeds the current limit and reject it before mapping search or AI inference begins.

## 4. Approved-mapping discovery

**FR-006 — Search before AI.** The system must search approved mappings before requesting an AI-generated mapping.

**FR-007 — Search order.** It must search the current user's approved mappings newest-to-oldest, followed by shared approved mappings newest-to-oldest. It must select the first compatible mapping found.

**FR-008 — Compatibility outcome.** Exact structural identity is not required. A mapping is compatible only when it can support a complete and unambiguous conversion to the current canonical Company model.

**FR-009 — Missing information.** A mapping is incompatible when information required by the canonical model is absent. The system must not fabricate missing source values.

**FR-010 — Additional source fields.** Extra source fields do not make an otherwise compatible mapping ineligible. They may remain unmapped.

## 5. AI-assisted mapping

**FR-011 — AI fallback.** If no approved mapping is compatible, the system must ask AI to propose a mapping to the canonical Company model.

**FR-012 — Unsuccessful proposal.** When AI cannot produce a complete and unambiguous mapping, the system must present its findings or failure reasons. It must not invent missing source data.

**FR-013 — Revision hints.** A user may request an AI revision by supplying written guidance about what should change. Direct editing of mapping rules is outside the v1 interaction.

**FR-014 — Retry limits.** The v1 demo must allow one initial AI mapping attempt and no more than two additional AI revision attempts within the same mapping workflow. The system must show remaining attempts and prevent additional inference after the limit is reached.

**FR-015 — Retry exhaustion.** At the retry limit, the user may confirm the latest proposal only if it is complete and unambiguous or discard the upload. An incomplete mapping may never be approved or used for transformation.

## 6. Human review and confirmation

**FR-016 — Review information.** Before transformation, the system must show:

- Source-to-canonical mapping rules
- A preview of transformed sample records
- Every source field that will remain unmapped
- The mapping version and canonical Company model version

This requirement applies to reused and AI-generated mappings. Unused source fields do not invalidate an otherwise complete mapping.

**FR-017 — Human control.** A mapping is valid only when every required field in the canonical `Company` model can be satisfied. No mapping may be used for transformation until the user confirms it.

**FR-018 — Review actions.** The user may confirm the mapping, reject and discard it, or request an AI revision with hints while attempts remain.

## 7. Mapping ownership and versioning

**FR-019 — Service ownership and reuse.** Every approved mapping definition is owned by the integration service provider and automatically becomes reusable across users.

**FR-020 — No proprietary content in shared definitions.** A reusable mapping definition must not contain source records, sample values, user prompts, user identity, or private audit history.

**FR-021 — Immediate reuse.** Each approved mapping version becomes immediately eligible for future searches.

**FR-022 — Version search.** Compatible versions are searched newest-to-oldest. Older versions remain searchable when a newer version does not match and remain retrievable during the defined mapping-retention period. After that period, an obsolete mapping definition may be deleted while historical audit retains its mapping ID/version and metadata.

**FR-023 — No ordinary activation state.** Normal mapping versioning does not require a separate active/inactive designation.

**FR-024 — Canonical-model binding.** Every mapping must record the canonical Company model version against which it was approved.

**FR-025 — Canonical-model change.** When the canonical Company model version changes, mappings from earlier model versions become ineligible until revalidated. Earlier mapping definitions remain subject to the retention behavior in FR-022.

Canonical-model administration and automated mapping revalidation are outside v1.

## 8. Transformation behavior

**FR-026 — Partial-conversion control.** Before processing begins, the user must be able to set `Allow partial conversion`. It defaults to Yes/On.

**FR-027 — Partial conversion enabled.** Valid records must be transformed and stored. Rejected records must be separated and assigned failure reasons.

**FR-028 — Partial conversion disabled.** If any record fails, the entire file run must fail and no transformed results from that run may be stored. The failure must still be audited.

**FR-029 — Results.** The user must be able to view the transformed results and the rejected records produced by a run.

## 9. Audit reporting and run status

**FR-030 — File-level audit.** Each run must record what was processed, when it ran, who initiated it, the result, the mapping ID/version reference used, the canonical-model version, and accepted/rejected record counts.

**FR-031 — Record-level audit.** Each record outcome must show its before-conversion fields, whether it was accepted or rejected, and the rejection reason when applicable. Proprietary record data remains private to the user's workspace and subject to retention limits.

**FR-032 — Run statuses.** Each run receives one of these immutable statuses:

- **Full Success/Complete:** every record was transformed and stored.
- **Partial Success/Complete:** some records were transformed and stored, while unresolved rejected records remain.
- **Total Failure:** no transformed records were stored. This includes whole-file rejection when partial conversion is off.

**FR-033 — Immutable run outcome.** Later recovery must not change the original run's status.

**FR-034 — Linked runs.** Reprocessing creates a new run linked to the original and receives its own audit record and status.

## 10. Rejected-record recovery

**FR-035 — Retention.** Rejected records must be retained until successfully resolved/reprocessed or 24 hours, whichever comes first. They remain subject to the demo-wide total-storage limit.

**FR-036 — Failure classification.** Before recovery, the system must classify a rejection as either potentially mapping-correctable or an unrecoverable source-data-quality failure.

**FR-037 — Direct export.** Missing, invalid, or otherwise unrecoverable source-data failures must bypass mapping search and AI and proceed to export.

**FR-038 — Alternate mapping.** For a mapping-correctable rejection, the system must first search approved mappings for another compatible candidate.

**FR-039 — AI reevaluation.** If no approved mapping resolves a mapping-correctable rejection, the user may request AI reevaluation within the configured usage limits.

**FR-040 — Recovery review.** Any alternate or AI-revised mapping must return to human review and confirmation before it is applied.

**FR-041 — Unresolved export.** If recovery is unsuccessful or declined, the user must be able to export the original rejected records in the same format in which they were uploaded.

## 11. Demo experience

**FR-042 — Downloadable sample.** The upload screen must provide a downloadable, editable synthetic CSV. It must not duplicate this with a separate in-app sample runner.

**FR-043 — JSON sample.** A downloadable JSON sample is included only if JSON support ships.

**FR-044 — Synthetic-data guidance.** The demo must tell visitors to use synthetic data and explain the supported format and record limit before upload.

**FR-045 — Guided proof.** The sample and demo state must make it possible to demonstrate both paths: AI proposal when no mapping matches and approved-mapping reuse on a later compatible upload.

**FR-046 — Exploration.** Visitors may edit the downloaded sample before uploading it to explore compatibility, unmapped-field notices, transformation failures, partial conversion, recovery, and export.

A running-conversion visualization is optional and must not block v1 completion.

## 12. Functional acceptance scenarios

Card #6 is functionally satisfied when the following behaviors can be demonstrated or tested:

1. A valid CSV passes intake validation; malformed, empty, and oversized files produce useful errors. An oversized file is rejected before mapping or AI work begins.
2. A compatible current-user mapping is found before shared mappings or AI are considered.
3. When no current-user mapping matches, a compatible shared mapping can be found without exposing another user's data.
4. When no approved mapping matches, AI proposes a mapping or explains why it cannot.
5. Reused and AI-generated mappings show rules, transformed samples, and unmapped fields before confirmation.
6. Transformation cannot begin without human confirmation.
7. A complete AI revision with user hints can be approved as a new reusable version; an incomplete mapping cannot be approved or executed.
8. A mapping associated with an older canonical-model version is not reused.
9. With partial conversion on, valid records are stored and rejected records are separated with reasons.
10. With partial conversion off, one failed record produces Total Failure and stores no transformed results.
11. A mapping-correctable rejection follows alternate-mapping search, bounded AI reevaluation, and human confirmation.
12. An unrecoverable source-data failure bypasses AI and can be exported in the original input format.
13. Reprocessing produces a linked run with its own status while preserving the original run's status.
14. Retry, record, retention, and usage limits are visible and enforced.

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
