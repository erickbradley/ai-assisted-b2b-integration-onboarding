# AI-Assisted B2B Integration Onboarding Platform
## Problem definition

Updated: 2026-09-13

This document describes the business problem and current v1 scope. The [v1 functional requirements](functional-requirements-v1.md) define the detailed behavior and acceptance scenarios.

### 1. Business context

B2B software platforms often need to ingest data from many customer or partner systems. Although those systems may represent similar business concepts, their source feeds frequently differ in schema structure, field names, nesting, data representation, and semantics.

Supporting a new source can therefore require specialized technical effort to:

- understand the incoming payload;
- determine what individual fields mean;
- map those fields to the platform's canonical model;
- identify missing or incompatible data;
- define required transformations and validation;
- test the resulting mapping;
- and determine whether the mapping is safe to use for runtime processing.

As the number and diversity of sources increase, this onboarding model can become expensive and difficult to scale.

The project is intentionally framed as a general B2B integration problem rather than being specific to logistics or another single industry.

---

### 2. Problem statement

Onboarding diverse customer source feeds requires too much specialized technical effort.

Each new source introduces schema and semantic differences that must be manually understood and translated into the receiving platform's canonical representation. Much of this analysis is repetitive but still requires technically capable personnel to inspect payloads, reason about field meaning, define mappings and transformations, and validate the result.

This creates three related business problems:

1. Integration capacity does not scale efficiently because supporting more sources requires corresponding increases in specialized effort.
2. Customer onboarding takes longer because source analysis and mapping are performed largely on a case-by-case basis.
3. Some lower-volume or unusual integrations may not be economically attractive to support.

---

### 3. Desired outcome

The platform should reduce the specialized effort required to onboard a new source feed while maintaining human control over decisions that affect runtime processing.

The intended business-value hierarchy is:

1. **Primary:** Support more customer integrations without proportional growth in specialized engineering effort or headcount.
2. **Secondary:** Reduce integration onboarding cycle time and improve customer time-to-value.
3. **Resulting benefit:** Make a broader set of integrations economically supportable.

The project will demonstrate the technical capability required to support this model. It will not claim to prove real-world staffing reductions or quantitative ROI.

---

### 4. Primary users and stakeholders

The primary business persona is a **Solution/Implementation Engineer** responsible for onboarding customer integrations. This includes people working under titles such as Solution Engineer, Integration Engineer, or Implementation Engineer.

The user is expected to understand source fields and data semantics, review proposed mappings and transformed previews, provide written revision guidance, and decide whether to confirm a mapping. Direct editing of mapping rules is outside the v1 interaction.

The public PoC represents this persona through a **demo user**, who uploads files, reviews and confirms mappings, inspects results, and exports rejected records.

The **integration service provider** owns approved mapping definitions, provides the canonical Company model, and establishes demo limits.

v1 uses lightweight demo identity. Enterprise registration, identity administration, authorization roles, and approval hierarchies are outside scope. Human confirmation is required, but there is no separate Approver role in v1.

Complex or unusual cases may still require deeper engineering expertise. The objective is to reduce how frequently such intervention is necessary, not eliminate engineering involvement entirely.

### 5. Primary product

The primary product capability is **source-data onboarding through reusable mappings**.

A representative processing flow is:

> **Company-data file → compatible approved mapping or AI proposal → human confirmation → deterministic conversion to the canonical Company model**

Mapping discovery precedes AI inference. The system searches the current user's approved mappings newest-to-oldest, then shared approved mappings newest-to-oldest, selecting the first compatible candidate.

A compatible mapping supports complete and unambiguous conversion to the current canonical model. Exact structural identity is not required, and extra source fields may remain unmapped. Missing information required by the model makes a mapping incompatible; the system must not fabricate source values.

Approved mappings are reusable across uploads and users rather than being limited to one source feed. AI assistance, audit reporting, and rejected-record recovery support this onboarding capability.

### 6. v1 source scope

v1 demonstrates onboarding through **manual uploads of company-data files**.

- CSV is required.
- JSON is a non-blocking stretch goal.
- YAML is deferred.
- Scheduled, API/SDK, and event-triggered ingestion are outside v1.

The demo rejects unsupported, malformed, empty, or oversized files with useful errors. The operator-configured record limit starts at 50 data records per file; a CSV header does not count. Oversized files are rejected before mapping search or AI inference.

An editable synthetic CSV provides the starting point for exploration. A JSON sample is provided only if JSON support ships. v1 does not require three predefined feeds or webhook sources.

### 7. Mapping and transformation scope

v1 demonstrates source-to-canonical field mapping and deterministic transformation governed by a human-confirmed mapping definition.

The review presents mapping rules, transformed sample records, unmapped source fields, and the mapping and canonical-model versions. Users request changes through written hints for AI revision rather than editing rules directly.

The detailed supported transformation operations and validation rules remain to be specified against the canonical Company model. Any permitted defaults must respect the requirement not to fabricate missing source values.

Preview computation supports review before confirmation. Applying a mapping to process and store a run requires human confirmation; the functional requirements should distinguish this preview from the confirmed processing run.

The project remains a bounded onboarding demonstration rather than a general-purpose ETL or arbitrary transformation-code platform.

### 8. Canonical data model

v1 converts company-data records into a **canonical Company model** supplied by the integration service provider.

Every mapping records the canonical-model version against which it was approved. When that version changes, mappings approved against earlier versions become ineligible until revalidated. Historical mappings remain available for audit.

Exact Company fields and validation rules remain design decisions. Canonical-model administration and automated mapping revalidation are outside v1.

The earlier generic event envelope and `IntegrationFailure` payload are superseded and are not v1 implementation targets.

### 9. AI-assisted onboarding workflow

A representative v1 workflow is:

1. The demo user uploads a company-data file.
2. The platform validates its format, data presence, and record count.
3. It searches approved mappings, checking the current user's mappings before shared mappings.
4. If no compatible mapping exists, AI proposes a mapping or explains why it cannot produce a complete and unambiguous proposal.
5. The user reviews mapping rules, transformed samples, unmapped fields, and version information.
6. The user confirms the mapping, discards the upload, or requests an AI revision with written hints while attempts remain.
7. Each approved mapping version becomes immediately reusable across users.
8. The platform processes the file deterministically using the confirmed mapping and the user's partial-conversion setting.
9. The user inspects results and audit information, then pursues bounded recovery or export for rejected records.

The platform enforces an operator-set AI retry limit and shows remaining attempts. At the limit, the user may confirm an acceptable latest proposal or discard the upload. The numerical limit depends on model choice, inference cost, and the demo cost ceiling.

AI recommends mapping configuration. It does not independently decide how individual records should be transformed during an approved run.

### 10. Mapping reuse and processing boundaries

Approved mapping definitions are owned by the integration service provider and automatically become reusable across users. Shared definitions must exclude source records, sample values, user prompts, user identity, and private audit history.

Compatible mapping versions remain searchable newest-to-oldest, including older versions when newer versions do not match. Normal versioning does not require a separate active/inactive state.

Each processing run must be deterministic and traceable to its confirmed mapping and canonical-model versions.

Before processing, the user controls **Allow partial conversion**, which defaults to On:

- With partial conversion enabled, valid records are transformed and stored; rejected records receive failure reasons.
- With partial conversion disabled, any record failure causes the whole run to fail, and no transformed results from that run are stored. The failure is still audited.

File-level audit information records what was processed, when, by whom, the mapping rules and versions used, and accepted/rejected counts. Record-level audit information preserves before-conversion fields, outcomes, and rejection reasons. Proprietary record data remains private to the user's workspace and subject to retention limits.

Each run has an immutable outcome: **Full Success/Complete**, **Partial Success/Complete**, or **Total Failure**. Later recovery creates a linked run with its own audit and outcome; it does not change the original run's status.

### 11. Bounded rejected-record recovery

v1 supports bounded recovery of rejected company-data records.

Before recovery, a rejection is classified as potentially mapping-correctable or an unrecoverable source-data-quality failure.

- Mapping-correctable failures first use a search for another compatible approved mapping.
- If no approved mapping resolves the failure, the user may request AI reevaluation within configured usage limits.
- Any alternate or AI-revised mapping returns to human review and confirmation before application.
- Unrecoverable source-data failures bypass mapping search and AI and proceed to export.
- If recovery fails or is declined, the user can export the original rejected records in their input format.

Rejected-record retention is bounded. Time and storage limits will be defined with nonfunctional requirements and the cost model.

This recovery flow replaces the earlier simulated downstream API replay demonstration. The primary business thesis remains reducing onboarding effort through mapping assistance and reuse.

### 12. Public demo model

The portfolio demonstration lets a reviewer explore the platform interactively without presenting the PoC as a production SaaS application.

The upload screen provides an editable synthetic CSV and explains the supported format, record limit, and synthetic-data-only guidance. Visitors download and optionally edit the sample, then upload it. There is no separate in-app sample runner.

The sample and demo state must demonstrate both AI proposal when no mapping matches and approved-mapping reuse on a later compatible upload. Visitors can also explore unmapped fields, transformation failures, partial conversion, recovery, and export.

Human review, version information, audit visibility, and private workspace data support the demonstration. Detailed identity, isolation, retention, and cost controls remain architecture and nonfunctional requirements work.

A running-conversion visualization is optional and must not block v1 completion.

### 13. v1 success definition

v1 succeeds when it demonstrates that:

> **Company-data files can be onboarded through approved-mapping reuse or AI-assisted proposals, reviewed and confirmed by a human, and converted deterministically into the canonical Company model with auditable outcomes and bounded recovery of rejected records.**

The functional requirements define the acceptance scenarios, including intake validation, mapping search order, human confirmation, version compatibility, partial conversion, linked recovery runs, export, and enforced limits.

This is a demonstrated-capability success criterion. The PoC does not claim a measured reduction in engineering effort, onboarding time, headcount, or production costs. Those remain business hypotheses for a real deployment to validate.

### 14. Explicit v1 exclusions

v1 is not a general-purpose integration platform, connector marketplace, ETL engine, autonomous integration agent, or production SaaS application.

The following are outside the current implementation:

- YAML support and JSON support if the stretch goal does not ship;
- scheduled, API/SDK, and event-triggered ingestion;
- direct user editing of mapping rules;
- enterprise registration, identity administration, roles, and approval hierarchies;
- canonical-model administration and automated mapping revalidation;
- production-scale storage, retention, and recovery controls;
- the former `IntegrationFailure` event-normalization and simulated remediation/replay workflow.

Human confirmation remains mandatory before processing a run. AI-generated mappings do not automatically enter processing use.

### 15. Assumptions

The current problem definition assumes:

- company-data CSV files can demonstrate meaningful schema and semantic differences;
- a canonical Company model provides a useful target for onboarding;
- a technically capable user can review mapping proposals and provide written revision guidance;
- approved-mapping reuse can reduce repeated analysis across compatible uploads;
- shared mapping definitions can be separated from private source data and audit history;
- human confirmation, mapping versioning, and deterministic processing are central to the demonstration;
- synthetic data and lightweight demo identity are appropriate for the public PoC;
- bounded file sizes, AI usage, and retention are necessary to keep the demo manageable.

### 16. Remaining unknowns

The following decisions remain for requirements refinement and architecture work:

- exact canonical Company fields and validation rules;
- supported deterministic transformation operations and permitted defaults;
- mapping representation and compatibility evaluation;
- how preview execution is distinguished from confirmed processing;
- rejection classification and alternate-mapping selection details;
- numerical AI retry and usage limits;
- retention duration and total-storage limits;
- lightweight identity and workspace isolation mechanisms;
- private metadata associations needed for current-user mapping search;
- audit storage, observability, reliability, and recovery targets;
- AI model, inference cost, and demo cost ceiling;
- AWS service selection;
- whether the JSON stretch goal will ship.

The initial 50-record limit, human confirmation, cross-user mapping reuse, partial-conversion behavior, and immutable run outcomes are already defined in the functional requirements.

### 17. Decision provenance and superseded scope

The original business framing and iterative design discussions established the project's enduring purpose: reduce specialized onboarding effort, retain human control, use deterministic versioned mappings, and demonstrate capability without unsupported ROI claims.

The finalized [v1 functional requirements](functional-requirements-v1.md), dated 2026-09-13, establish the current implementation scope. This problem definition has been updated to reflect that scope.

The earlier problem definition recorded JSON-only webhook inputs, three fictional feeds, an `IntegrationFailure` canonical payload, direct mapping edits, an Approver role, and simulated downstream replay. Those decisions are superseded for v1 and are retained here only as historical context.

The current scope uses CSV-first company-data uploads, a canonical Company model, approved-mapping discovery before AI, written revision hints, demo-user confirmation, shared reusable mappings, and bounded rejected-record recovery.

The assistant's role is to structure and document the selected scope. Open architecture questions remain explicit rather than being presented as settled requirements.
