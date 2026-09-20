# AI-Assisted B2B Integration Onboarding Platform
## Nonfunctional Requirements & Constraints

**Status:** v1 portfolio/demo requirements  
**Scope:** Company-data file onboarding and transformation platform  
**Primary source format:** CSV  
**Stretch format:** JSON  
**Deferred format:** YAML  
**Demo processing limit:** First 50 data records per uploaded file

---

## 1. Purpose

This document defines the nonfunctional requirements and constraints for v1 of the AI-Assisted B2B Integration Onboarding Platform.

The requirements intentionally distinguish between:

1. **Enterprise-informed architectural requirements** — qualities the design should account for when handling potentially proprietary enterprise data.
2. **Portfolio/demo implementation constraints** — controls appropriate to a self-funded, synthetic-data demonstration.
3. **Production evolution** — capabilities that may be required in a true enterprise implementation but are intentionally outside v1.

The objective is to demonstrate sound architecture judgment without representing the portfolio system as a production-grade enterprise SaaS platform.

---

# 2. Governing Principles

## NFR-2.1 — Enterprise-informed data posture

Although the public demo will permit only synthetic data, the architecture must treat uploaded source data as though it could contain proprietary enterprise information.

Production-grade controls do not all have to be implemented in v1, but omitted controls must be explicitly treated as deferred rather than assumed unnecessary.

## NFR-2.2 — Data minimization

The platform should retain, transmit, log, and expose only data required for a defined purpose.

Data must not be retained “just in case.” If a future need for additional retained data is demonstrated, retention should be limited to the specific data required and only for as long as necessary.

## NFR-2.3 — Human control

AI is assistive.

No AI-generated mapping may become executable or reusable without explicit human approval.

---

# 3. Security and Trust Boundaries

## NFR-3.1 — Workspace isolation

v1 must enforce meaningful logical isolation between demo workspaces.

A user operating within one workspace must not be able through supported application or API behavior to access another workspace's:

- uploaded source files;
- transformed results;
- rejected records;
- processing runs;
- workspace-specific artifacts.

v1 is not required to implement enterprise-grade tenant administration, federation, identity lifecycle management, or sophisticated RBAC.

### Validation

Create at least two workspaces and verify that actions scoped to one cannot retrieve or modify data belonging to the other.

---

## NFR-3.2 — AI minimum-necessary exposure

AI operations must follow a **minimum-necessary data exposure principle**.

The system must define maximum boundaries for what each AI operation may receive.

The minimum required information may vary by task, but the platform must not expose additional source information merely for convenience.

Some increases in AI-visible data may require explicit user authorization.

---

## NFR-3.3 — Responsibility for AI data appropriateness

The user/organization has primary responsibility for ensuring that information submitted for AI-assisted processing is appropriate and limited to what is necessary.

The platform may implement safeguards intended to identify potentially sensitive information, but such safeguards are defense-in-depth controls and must not be represented as guaranteeing detection of PII, protected information, or other sensitive data.

---

## NFR-3.4 — Sensitive-data warning and override

If a platform safeguard identifies potentially sensitive data before additional AI exposure:

1. AI processing must pause.
2. The user must be shown that potentially sensitive data was detected.
3. Processing may continue only after explicit user acknowledgement/override.

For v1, any authenticated user in the relevant workspace may perform the override.

This is a deliberate demo simplification and must not be represented as enterprise authorization governance.

---

## NFR-3.5 — AI interaction retention

The platform must not retain the actual AI input/context used to generate or revise a mapping after the AI interaction is complete.

This includes source values or user prompts supplied specifically as AI context.

Relevant provenance metadata may be retained separately.

---

## NFR-3.6 — Encryption

All application data must be encrypted:

- **in transit**; and
- **at rest** wherever persisted.

This applies to business data as well as application-owned persistent data.

---

## NFR-3.7 — Secrets and credentials

Application secrets, API keys, and credentials must:

- not be hard-coded;
- not be committed to source control;
- not be stored in plaintext application configuration.

v1 should use an AWS-native managed secrets capability where secrets are required.

Exact AWS service selection remains an architecture decision.

---

## NFR-3.8 — Logging and sensitive data

Application logs must not contain:

- source record values;
- transformed record values;
- rejected-record contents.

Logs may contain:

- identifiers;
- statuses;
- counts;
- error codes;
- timings;
- technical metadata necessary for troubleshooting.

---

# 4. Reliability and Failure Behavior

## NFR-4.1 — Idempotent processing requests

Accidental duplicate processing must be prevented using a unique processing-request identity.

Submitting the same processing request more than once must not create multiple independent processing runs.

Identical source content may still be intentionally processed again under a new request identity.

---

## NFR-4.2 — No automatic processing retries

v1 will not automatically retry failed processing attempts.

A failed run must become visible to the user.

The user determines whether another attempt should occur.

A production implementation may later introduce bounded automatic retry behavior for appropriately classified transient failures.

---

## NFR-4.3 — Retry creates a new run

Every user-initiated retry after failure must create a **new processing run linked to the original**.

The original failed run must remain unchanged.

This same principle applies to intentional reprocessing.

---

## NFR-4.4 — User-controlled partial conversion

Each processing run must allow the user to choose whether partial conversion is permitted.

### Partial conversion enabled

- Successfully transformable records may proceed.
- Records that cannot be transformed must be classified as rejected records.
- Rejected records must include an understandable rejection classification/reason.

### Partial conversion disabled

The run requires **all-or-nothing delivery**.

If any record fails the required validation or transformation, no transformed records from that run may be delivered to the target system.

How this guarantee is technically implemented is deferred to architecture design.

---

## NFR-4.5 — Failed-run cleanup

Persisted temporary processing artifacts that have no demonstrated continuing diagnostic or business purpose must be cleaned up after:

1. the failed run has been recorded; and
2. required diagnostic information has been captured.

The system must not retain intermediate processing data merely because it may theoretically be useful later.

---

# 5. Mapping Validity and AI Failure Behavior

## NFR-5.1 — Mapping completeness

A mapping is valid only when **every required field in the canonical `Company` model can be satisfied**.

A valid mapping does not require every source field to be consumed.

---

## NFR-5.2 — Unused source-field visibility

Before approval, the user must be able to see which source fields are unused/unmapped.

Unused source fields do not invalidate an otherwise complete mapping.

They must not, however, be silently ignored during review.

---

## NFR-5.3 — Incomplete AI mapping behavior

An incomplete AI-generated mapping is never valid for approval or execution.

If AI cannot produce a complete mapping, the platform may:

- display fields successfully mapped;
- identify unresolved canonical fields;
- identify relevant unmapped information;
- explain why a complete mapping could not be established;
- suggest, where possible, additional context or hints the user might supply.

The user must provide additional information/hints and explicitly request another AI attempt.

Only a subsequently complete mapping may proceed to approval.

---

## NFR-5.4 — AI mapping retry limit

After the initial AI mapping attempt, the demo will permit no more than **two additional AI revision attempts** within the same mapping workflow.

This is explicitly a self-funded demo constraint.

The exact number may be adjusted after real inference costs are measured, but AI retries must remain bounded.

---

# 6. Observability and Auditability

## NFR-6.1 — Unified audit stream

The platform must maintain one combined audit stream for:

- mapping lifecycle actions;
- approvals;
- AI involvement;
- processing runs;
- failures;
- sensitive-data overrides;
- retries/reprocessing;
- other material system actions.

Consumers may distinguish actions using event type and metadata.

---

## NFR-6.2 — Append-only audit history

Audit history must be append-only through normal application behavior.

Existing events must not be modified or deleted.

Corrections must be represented through additional events rather than rewriting historical events.

---

## NFR-6.3 — File/run-level audit information

A processing run should retain sufficient provenance to identify:

- what occurred;
- when it occurred;
- the responsible user/actor;
- processing result;
- mapping ID and version;
- canonical-model version;
- number of records processed;
- number accepted/transformed;
- number rejected;
- failure classifications;
- links between original and reprocessed/retried runs.

---

## NFR-6.4 — Record-level review visibility

During the active review/retention window, an authorized workspace user must be able to inspect record-level outcomes including:

- fields before conversion;
- whether the record succeeded or was rejected;
- rejection reason where applicable.

After record-level business data expires, this detail must no longer be retained.

---

## NFR-6.5 — AI provenance

For mappings produced with AI assistance, provenance must identify:

- that AI assistance occurred;
- the resulting mapping version;
- the subsequent human approval.

The platform does not need to retain every AI revision attempt or the original AI prompt/context.

---

## NFR-6.6 — Mapping provenance

Historical runs must reference the mapping ID/version used.

The run does not need to contain a duplicate snapshot of the mapping rules as long as the referenced mapping version remains retrievable during its applicable retention period.

---

## NFR-6.7 — Operational metrics

v1 must expose sufficient metrics to understand both system health and demo cost behavior.

Metrics should include at minimum:

- service health;
- processing/request counts;
- success/failure counts;
- processing latency;
- AI invocation counts;
- storage consumption;
- usage/quota consumption.

Product-analytics instrumentation beyond these operational needs is outside the v1 NFR scope.

---

## NFR-6.8 — Alerting

Alerts should prioritize conditions indicating either:

1. the system is materially broken for a user; or
2. the owner's cost exposure may be exceeding expected limits.

Examples include:

- service-level failures;
- significant run failures;
- abnormal AI usage;
- abnormal storage growth;
- unexpected request volume;
- owner-defined cost threshold conditions.

Routine application activity should remain visible through logs/metrics rather than creating unnecessary alerts.

---

# 7. Data Lifecycle and Retention

## NFR-7.1 — Downstream data ownership

Long-term persistence of successfully transformed business data belongs to the downstream target enterprise system.

The integration utility is not intended to become the authoritative system of record for integrated business data.

---

## NFR-7.2 — Uploaded source files

Uploaded source files must be short-lived.

They may be retained only long enough to support:

- processing;
- review;
- immediate troubleshooting.

The initial v1 retention window is **24 hours**.

The value may be reduced after launch based on observed usage and cost.

---

## NFR-7.3 — Transformed results

The integration utility may retain its local copy of transformed results for the same short post-processing review window.

The initial review window is **24 hours**.

After expiration, the utility's copy must be deleted even though the corresponding business data may continue to exist in the downstream target system.

---

## NFR-7.4 — Rejected records

Rejected records must be retained until the earlier of:

1. successful reprocessing/resolution; or
2. the **24-hour** review-window expiration.

---

## NFR-7.5 — Historical information after business-data deletion

After source, transformed, and rejected record contents expire, the platform must retain:

- audit/provenance metadata;
- aggregate processing counts;
- outcome/status information;
- mapping/canonical version references;
- failure classifications;
- reprocessing relationships.

Expired business-record values must not remain in historical records.

---

## NFR-7.6 — Mapping definition lifecycle

Approved mappings are service-owned, reusable, versioned, and bound to a canonical-model version.

Superseded mapping versions must remain retrievable for a predefined retention period.

For the demo, once that retention period expires, an obsolete mapping definition may be deleted even if older audit history still references its mapping ID/version.

The exact mapping-retention duration remains open.

Production systems may instead require archival or retention for as long as historical runs depend on the mapping.

---

## NFR-7.7 — Total demo storage ceiling

v1 must enforce a hard total-storage cap across the public demo.

The numeric cap will be determined during architecture/cost modeling.

When the limit is reached, storage recovery must occur in this order:

1. Delete already-expired retained data.
2. If necessary, delete the oldest unexpired retained run data.
3. Only if sufficient space cannot be recovered, reject new uploads.

This is explicitly a self-funded demo policy and not the intended default for an enterprise implementation.

---

## NFR-7.8 — Early-retention eviction visibility

If unexpired run data is removed early because the demo-wide storage ceiling was reached:

- the historical run/audit record must remain;
- the UI must clearly indicate that detailed data was removed early due to the public demo's storage constraints.

---

# 8. Cost and Abuse Controls

## NFR-8.1 — Cost model

Demo cost requirements must account for both:

- overall monthly spend; and
- approximate per-run processing cost.

The project should understand the economics of a representative onboarding/processing run rather than relying solely on aggregate monthly billing.

---

## NFR-8.2 — Idle versus active cost

Separate expectations must be established for:

- **idle periods**, when the system receives little or no use; and
- **active demo/testing periods**.

Because the system is expected to be idle most of the time, idle operating cost should be kept deliberately low.

Exact dollar targets remain open pending AWS cost modeling.

---

## NFR-8.3 — Layered consumption limits

Cost protection should operate at multiple levels:

1. bounded AI usage within an individual workflow;
2. broader per-user/workspace usage limits to discourage abusive, reckless, or repeated consumption;
3. an owner-level demo-wide financial protection threshold.

The intent is to prevent abusive or careless public usage without unnecessarily constraining legitimate portfolio exploration.

Exact quota values and reset periods remain open.

---

## NFR-8.4 — Owner financial protection threshold

If the owner-level financial protection threshold is reached, the **cost-generating interactive demo must stop accepting further use**.

Protecting the owner from uncontrolled financial exposure takes precedence over demo availability.

This does not require the static portfolio site, documentation, or other non-cost-generating material to become unavailable.

---

## NFR-8.5 — Recovery from consumption limits

Routine user/workspace usage quotas should reset automatically after their defined period.

By contrast, reaching the owner-level financial protection threshold must require **manual owner intervention** before cost-generating interactive use can resume.

The exact AWS/application mechanism will be determined during architecture design.

---

# 9. Performance, Availability, and Recovery

## NFR-9.1 — Demo processing experience

For valid demo uploads of no more than 50 records, processing should normally complete within an **interactive wait experience**.

The evaluator should not normally need to leave the workflow and return later.

This is intentionally different from a production enterprise implementation, which may reasonably require asynchronous processing for large or long-running workloads.

---

## NFR-9.2 — Processing timeout

If processing exceeds the defined interactive-response target:

- processing must fail gracefully;
- the run must enter a clear failed state;
- hidden background processing must not continue;
- the user may initiate a new linked retry.

The exact timeout value remains open.

---

## NFR-9.3 — Availability

v1 has **no formal uptime SLA or percentage-availability target**.

The system should be reasonably reliable and recoverable, but production-grade availability is outside v1.

---

## NFR-9.4 — Recovery Time Objective

The demo targets a **24-hour RTO** for restoration after a service-impacting failure.

This is a best-effort portfolio target, not a contractual service commitment.

Recovery may take longer when manual owner intervention is required and the owner is unavailable.

---

## NFR-9.5 — Tiered Recovery Point Objectives

Recovery requirements must reflect the value and reproducibility of different data categories.

### Durable application data

Approved mappings and audit/provenance records initially target an **RPO of 1 hour**.

This target may be relaxed if implementation complexity or cost proves disproportionate for the portfolio system.

### Transient run data

The following have **no formal RPO**:

- uploaded source files;
- transformed results;
- rejected records;
- in-progress run state.

Loss of transient data may require the user to initiate another run.

---

# 10. Maintainability and Operability

## NFR-10.1 — Reproducible lifecycle

v1 deployment, configuration, update, and teardown should be repeatable and documented.

The system should minimize undocumented manual configuration or one-off environment steps.

---

## NFR-10.2 — Diagnosability

Representative failures should be reasonably diagnosable using:

- logs;
- metrics;
- audit history;
- alerts;
- documented troubleshooting guidance.

The requirement is portfolio-appropriate operability, not production-grade SRE maturity.

---

## NFR-10.3 — Rebuildability

Where practical, infrastructure and configuration should be sufficiently reproducible that the environment can be torn down and recreated without relying on undocumented manual reconstruction.

Exact implementation mechanisms are architecture decisions.

---

# 11. Explicit Demo Constraints

The following constraints are intentionally specific to the public/self-funded demonstration:

- Synthetic data only.
- Manual data upload.
- CSV required for v1.
- JSON is a non-blocking stretch goal.
- YAML deferred.
- Maximum of the first 50 records per upload.
- AI mapping attempts are bounded.
- Short-lived business-data retention.
- Global storage cap.
- Per-user/workspace usage controls.
- Owner-level financial shutdown control.
- No formal uptime SLA.
- Interactive rather than long-running asynchronous processing.
- Lightweight identity and authorization.
- Potential early eviction of retained run data under storage pressure.

These constraints must not be presented as universally appropriate production-enterprise behavior.

---

# 12. Production Evolution / Explicitly Deferred

Potential enterprise evolution includes, but is not limited to:

- enterprise identity lifecycle management;
- richer RBAC and separation of duties;
- category-specific sensitive-data authorization;
- stronger tenant administration;
- automated mapping revalidation;
- configurable customer retention policies;
- long-term mapping archives;
- production-grade retry policies;
- asynchronous/batch processing;
- tighter or differentiated RTO/RPO targets;
- formal availability/SLA targets;
- enterprise-scale storage policies;
- larger upload limits;
- customer-specific cost/usage governance;
- compliance-driven retention;
- more sophisticated threat and abuse controls.

These are not required to prove the v1 architecture thesis.

---

# 13. Open Quantitative Parameters

The following requirements are conceptually decided but still require numeric values during architecture/cost modeling or implementation testing:

| Parameter | Current status |
|---|---|
| Monthly demo cost ceiling | Open |
| Idle-month cost target | Open |
| Active-demo cost ceiling | Open |
| Approximate/permitted cost per run | Open |
| Broader user/workspace usage quota | Open |
| Usage quota reset period | Open |
| Owner-level financial shutdown threshold | Open |
| Hard total-storage ceiling | Open |
| Interactive processing timeout | Open |
| Mapping-version retention duration | Open |
| Final AI retry limit | Initial value = 2 revisions; revisit after cost measurement |
| Durable-data RPO | Initial target = 1 hour; revisit if disproportionate |
| Source/result/rejected-record retention | Initial value = 24 hours; may be reduced based on usage |

---

# 14. v1 NFR Thesis

v1 should demonstrate an architecture that is:

- secure enough to show credible enterprise trust-boundary thinking;
- deterministic and human-controlled where AI influences mappings;
- traceable through versioned mappings and append-only provenance;
- explicit about failure behavior and reprocessing;
- disciplined about data minimization and retention;
- observable enough to diagnose failures and understand cost;
- financially bounded for a self-funded public demonstration;
- recoverable without pretending to provide production-grade availability;
- reproducible and operable as a portfolio artifact.

The goal is not to make the demo enterprise-grade.

The goal is to demonstrate that the architecture decisions, constraints, and tradeoffs were made with **enterprise-grade reasoning**.
