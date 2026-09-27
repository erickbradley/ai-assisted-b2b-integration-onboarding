# AI-Assisted B2B Integration Onboarding Platform

## Application profile — pre-build

Status: Pre-build discussion baseline; all ten sections populated, with unresolved items explicitly retained. Not an approved architecture or completed implementation.

Started: 2026-09-27

This document describes the workload and system characteristics needed before architecture selection and ADRs. It is developed progressively with Erick, one section and one question at a time. Unanswered sections are not agreed decisions.

### Evidence labels

- **Known:** Established fact supported by a named source or direct evidence; does not imply that a planned capability is implemented.
- **Requirement:** Behavior or constraint the system must satisfy.
- **Estimate:** Provisional quantity or range, with its basis recorded.
- **Assumption:** Working premise that needs validation.
- **Unknown:** Information or a decision that remains unresolved.

### Source tracking

The following repository documents form the current, mutually aligned source set. Git history preserves earlier revisions, so this profile does not identify individual Git file objects.

- [Problem definition](problem-definition.md): defines the business problem, current v1 scope, assumptions, and exclusions.
- [Functional requirements](functional-requirements-v1.md): defines required behavior and acceptance scenarios.
- [Nonfunctional requirements](nonfunctional-requirements-v1.md): defines quality attributes, constraints, retention, security, reliability, cost, and recovery expectations.

The profile was also informed by external project artifacts retained outside this repository:

- Scope decisions and an XMind workflow extraction dated 2026-09-12. These record the move to file onboarding and a canonical Company model; superseded `IntegrationFailure` material is historical only.
- The [Trello project plan](https://trello.com/b/PdtLdWkP/ai-assisted-b2b-integration-onboarding-platform-project-plan), reviewed 2026-09-27. Card #22 defines this profile; card #10 includes the canonical Company contract as unfinished design work.
- The BeSA learning record, used as supporting learning context rather than as a source of project requirements.

Newer explicitly approved decisions take precedence over superseded source material. Any future conflict among the repository documents must be reconciled before the affected statement is treated as settled.

## 1. Purpose and scope

**Requirement — Document boundary:** Keep this profile architecture-neutral. Describe the workload, constraints, and system characteristics before selecting architecture, AWS services, or recording ADRs.

**Requirement — Depth:** Establish a useful pre-build baseline for architecture comparison, feasibility validation, and later post-build variance analysis. Prioritize material design inputs over exhaustive detail.

**Known — Business purpose:** The finalized problem definition identifies reducing specialized effort for B2B source-data onboarding as the primary purpose, with faster onboarding and broader economically supportable integration coverage as intended benefits. v1 demonstrates capability rather than claiming measured staffing savings or ROI.

**Requirement — Primary profile scope:** Use a hybrid profile: the planned v1 application and demo-scale workload form the design baseline. Explicitly record production evolution considerations where they materially affect later architecture decisions. This describes the intended system before implementation, not evidence of an already implemented capability.

**Requirement — Production scope boundary:** Production evolution considerations are observations, not v1 requirements. They do not expand v1 scope unless Erick explicitly promotes them into requirements. Keep these considerations separate from the baseline and label their underlying statements as Known, Requirement, Estimate, Assumption, or Unknown as appropriate; a hypothetical production need must not be presented as an established requirement.

Decision provenance: Erick selected option D and accepted the production scope boundary in this working session.

**Requirement — Demonstrated capability:** A user can provide a sample dataset in a supported format, have the application process it, and receive the appropriate outcome for the scenario, as defined in the finalized functional requirements. CSV is required for v1; JSON remains a non-blocking stretch goal. This profile summarizes that capability without duplicating the detailed functional flows.

**Requirement — Architecture visibility:** The portfolio experience must give the user an opportunity to see and understand the underlying solution architecture. This is a requirement for explaining the eventual solution; it does not select an architecture in this pre-build profile. The presentation mechanism remains unresolved.

Decision provenance: Erick supplied the demonstrated capability and architecture-visibility intent in this working session. He clarified that JSON was an example, confirming the existing format baseline: CSV required, JSON optional.

## 2. Critical workflows

**Requirement — Core workflow summary:** Upload and validate a sample dataset; search approved mappings before requesting an AI proposal; present mapping rules, transformed sample records, and unmapped fields for human review; apply the confirmed mapping deterministically; display results and audit outcomes. Rejected records follow the bounded recovery/export flow. Source: finalized functional requirements, FR-001–FR-041. This is a summary, not a replacement for the detailed flow.

**Requirement — Demonstration priority:** Beyond the successful first-upload path, emphasize human-guided AI revision: the user reviews a proposal, supplies written correction hints, reviews the revised proposal, and confirms it before transformation. Existing retry and usage limits apply. Source: Erick's selection of scenario B; FR-013–FR-018. This priority does not remove mapping reuse or rejected-record recovery from v1.

**Requirement — Finalized AI limits and completeness:** One initial AI mapping attempt plus no more than two revisions per mapping workflow; incomplete mappings cannot be approved or executed (current FR-014–FR-017; NFR-5.1–5.4). Improving coverage alone does not permit approval while required canonical fields remain unresolved.

**Known — Portfolio intent:** Erick selected this scenario to make his applied AI capabilities visible to prospective employers.

**Assumption — Career relevance:** A credible demonstration of human-guided AI integration will strengthen positioning for the targeted architecture roles. Its hiring impact remains unvalidated; job-posting interest alone does not establish that impact.

**Requirement — Evidence of a useful revision:** For the selected correction scenario, a reviewer should be able to observe more successfully mapped fields after the AI revision than before it. This is a demonstration success criterion, not a guarantee that every retry improves the mapping. Source: Erick's answer in this working session.

**Assumption — Correction quality:** The user supplies valid corrections relevant to the mapping problem. This premise does not establish that the AI will apply them correctly.

**Requirement — Review evidence for a useful revision:** The existing review flow provides the evidence: the user inspects the revised source-to-canonical rules, transformed sample-record preview, and unmapped-field list, then confirms whether the proposal reflects the intended correction before an actual processing run (FR-013, FR-016–FR-018). After processing, transformed results and file/record audit outcomes provide further evidence of the applied mapping and any failures (FR-029–FR-031). No separate correctness-review workflow is added by this profile.

**Requirement — Preview boundary:** Before confirmation, the system may evaluate a proposed mapping only against a bounded sample to produce the required preview. Preview evaluation must not create a processing run, store successful business results, or make the mapping reusable. Human confirmation remains required before actual run processing and result persistence (FR-016–FR-018; NFR-2.3).

**Requirement — Mapping constraints:** A complete, unambiguous mapping must not invent missing source values; extra source fields may legitimately remain unmapped (FR-008–FR-012). Increased field coverage is an indicator for the chosen demonstration scenario, not independent proof of semantic correctness. Human review of rules and sample results is the specified confirmation mechanism; approval or successful processing does not guarantee that every mapping is semantically correct.

## 3. Actors and usage profile

**Requirement — User responsibilities:** The demo user uploads sample data, reviews and confirms mapping proposals, requests AI revisions when needed, inspects results, and exports rejected records. The integration service provider owns approved mapping definitions, establishes demo limits, and provides the canonical Company model. Source: finalized functional requirements, Section 2. These are responsibilities, not a choice of identity or access implementation.

**Requirement — Modes of use:** Support both independent visitor exploration and Erick-led demonstrations. A visitor must be able to explore the core demonstration without Erick present; a guided walkthrough supports interview follow-up and deeper exploration. Source: Erick's selection and clarification in this working session.

**Assumption — Likely visitor journey:** Independent exploration by a hiring manager or technical reviewer is the more likely initial path. A guided demonstration is more likely to follow if an interview occurs. This is an expected usage pattern, not measured visitor behavior.

**Estimate — Independent visit duration:** Approximately 5–10 minutes, based on Erick's hoped-for visitor engagement rather than observed usage. This is an attention-budget estimate, not a system response-time requirement.

**Assumption — Direction of uncertainty:** If the 5–10 minute estimate is wrong, visits are more likely to be shorter than longer. Do not rely on every visitor completing an extended walkthrough to understand the project.

**Unknown — Observed engagement:** Actual visit duration and completion of the core demonstration remain unmeasured.

**Requirement — v1 visit continuity boundary:** Treat each visit as a fresh exploration. Returning visitors do not need to resume prior work. This is an intentional demo/PoC concession for v1. It does not remove in-visit review and recovery, approved-mapping reuse, or existing audit requirements, and does not establish a data-deletion schedule. Source: Erick's selection and clarification in this working session.

**Requirement — Temporary identity scope:** After human verification, assign the visitor a temporary session-level demo identity and logically isolated workspace without account registration. For v1, “current user” means this temporary identity. It scopes private run data, usage limits, sensitive-data assertions, and current-user mapping-search precedence during the session. It does not make approved mapping definitions user-owned, create cross-visit continuity, or require the visitor to recover the same workspace later.

**Unknown — Production evolution: continuity:** Evaluate cross-visit continuity as a possible enterprise/production requirement, particularly for long-running, mission-critical, or recurring workloads. It is not a v1 requirement or an agreed production design. Visitor return/resume behavior and uninterrupted workload execution are related but distinct needs to assess during that evaluation.

## 4. Data and state

**Requirement — Sample-data guidance:** Tell visitors to use synthetic data. Visitors may edit the supplied sample or provide their own synthetic dataset within the supported formats and demo limits. Source: FR-042–FR-046 and Erick's clarification in this working session.

**Requirement — Demo-use acknowledgment:** Before uploading, visitors must acknowledge that this is a demo system intended only for synthetic data and that it is not intended or designed to handle proprietary, private, sensitive, regulated, or otherwise protected information. This acknowledgment records the visitor's assertion and does not verify the dataset's contents. Source: Erick's clarification in this working session.

**Requirement — Suspected-sensitive-data safeguard:** Where the platform can reasonably detect potentially private or sensitive data, pause further AI-assisted processing and explain the concern. The current demo user may continue only by explicitly asserting that the detection is a false positive and the submitted data contains no prohibited private or sensitive information. The override does not authorize prohibited data, prove the data is safe, or replace the preventative acknowledgment. Detection remains best-effort and must not be presented as guaranteed (FR-044–FR-044A; NFR-3.3–3.4).

**Known — Assurance limit:** Guidance and acknowledgment do not establish that uploaded data is synthetic or free of restricted information. The profile must not treat compliance as verified.

**Requirement — Submitted work survives visitor departure:** Closing the page must not cancel already-submitted processing. The system must continue that work to its applicable success or failure outcome within existing usage limits and record the required results and audit outcomes. This does not promise successful processing in the presence of errors or require returning visitors to recover the prior session. Source: Erick's selection in this working session.

**Requirement — Human approval remains mandatory:** Completing already-submitted work does not authorize the system to cross a pending human-confirmation boundary. A proposal awaiting review remains subject to FR-017; visitor departure is not approval to transform data.

**Requirement — Session-level accumulation:** Transformed Company records may accumulate across processing runs within the current demo session, subject to applicable retention periods and usage/storage limits. This is a session-scoped dataset, not a dataset shared across visitors. It does not require cross-visit continuity or expanded user administration. Source: Erick's selection of accumulation with an explicit session-level boundary in this working session.

**Assumption — Production evolution: organizational sharing boundaries:** Session-scoped data is a demo/PoC concession. A production application should distinguish mappings and data by organization, department, or team, allowing mapping reuse to be restricted to a team or broadened to the organization. This describes scope and visibility, not a requirement for long-term business-data retention by the integration application. It does not change v1 service-owned, cross-user mapping reuse.

**Unknown — Production evolution: sharing policy details:** The exact hierarchy, who can select or change reuse scope, and the relationship between mapping-sharing permissions and business-data access remain unresolved. Sharing a mapping does not by itself authorize sharing the underlying data. No enterprise administration design is selected here.

**Requirement — Established retention baseline:** Uploaded source files and local transformed results have an initial 24-hour review/retention window. Rejected records remain until successful resolution/reprocessing or 24 hours, whichever comes first. The 24-hour limit is specifically a demo/PoC concession, not a production retention requirement. Historical audit keeps metadata, not expired business-record values. Under storage pressure, delete expired data first, then oldest unexpired run data, and reject uploads only if space still cannot be recovered; indicate early eviction to the user. The numeric storage ceiling and mapping-definition retention duration remain open (NFR-7.2–7.8). Closing a page alone does not establish the deletion schedule.

**Requirement — Audit versus review data:** Permanent audit/provenance is append-only metadata: identifiers, events, timestamps, actors, statuses, counts, failure classifications, mapping and canonical-model version references, and reprocessing relationships. Before-conversion fields, transformed values, and rejected-record contents are temporary record-review data available only during their retention window. Deleting those business values does not modify the historical audit record, which must not retain expired values (FR-030–FR-034; NFR-6.1–6.6, NFR-7.5).

**Requirement — Processing completion boundary:** Visitor departure does not cancel submitted work, but the separately defined hard processing timeout still applies: processing that reaches that timeout must fail visibly without continuing hidden background work. No automatic processing retry is required (NFR-4.2; NFR-9.2). The numeric timeout remains open.

**Requirement — Demo ownership boundary:** For v1, stored and displayed canonical results in the demo's canonical data store represent successful downstream delivery and serve as the theoretical system of record during the demo retention window. No separate target application is required. Organizational mapping/data visibility is a separate concern and does not change this ownership boundary.

**Assumption — Production evolution: downstream ownership:** In a production enterprise implementation, the target company's canonical application or data stores receive the transformed data and become its authoritative system of record. The integration utility may keep only a separate review or troubleshooting copy under its own retention policy. Validate this boundary against the actual enterprise context rather than treating it as a v1 requirement.

**Requirement — Retained-result lifecycle:** The v1 canonical results are deleted according to the demo retention policy even though they represent its theoretical system of record; this is a demo/PoC concession.

**Assumption — Production evolution: retention policy:** The short, initially 24-hour retention window is a demo/PoC concession, not a universal enterprise default. The production target system would retain delivered business data under its applicable policies, while any integration-utility review copy would follow a separate retention policy. Those policies must be defined against applicable legal, contractual, and operational obligations. More robust retention does not automatically mean retaining every data category longer or indefinitely.

**Unknown — Production retention obligations:** Applicable obligations, category-specific periods, deletion rules, and any preservation exceptions have not been established. Determine them for the actual enterprise context; do not infer concrete legal requirements or extend the v1 retention window here.

**Assumption — Repeated company records, provisional v1 baseline:** Retain separate transformed results associated with each processing run, even when records may describe the same company. Do not assume that later uploads update an authoritative current company record. Erick selected this behavior for now, subject to the definition of the canonical target data model. Session-level accumulation therefore does not presently require cross-run company matching, deduplication, or merge rules.

**Unknown — Canonical-model dependency:** Revisit the provisional repeated-record behavior when the canonical Company model's identity and record semantics are defined. A company identifier alone does not establish whether later records replace earlier ones; the intended processing behavior also needs to be explicit. Original run audit outcomes remain immutable under FR-033–FR-034 regardless of that later decision.

**Known — Canonical-model definition status, checked 2026-09-27:** The current repository contains the problem definition and functional/nonfunctional requirements, but no canonical schema artifact. The current problem definition explicitly leaves exact Company fields and validation rules as design decisions (Sections 8 and 16). Trello's unfinished [Milestone 4 — Produce detailed architecture design](https://trello.com/c/vJayBxZn/10-milestone-4-produce-detailed-architecture-design), in Later — Architecture & Design, includes the canonical Company contract. No standalone canonical-model-definition card was found among the returned board cards. Related [Milestone 3](https://trello.com/c/2PkfqddG/9-milestone-3-compare-architecture-options-write-adrs) includes model storage/versioning and mapping invalidation. The profile therefore retains model details as unresolved rather than defining them here.

**Requirement — Repeated request versus repeated content:** The provisional append-per-run behavior applies to intentionally new requests. An accidentally resubmitted request must not create another independent run; identical source content may be intentionally processed again with a new request identity (NFR-4.1). This is separate from company-level matching or deduplication.

**Requirement — Mapping-definition retention:** Retain superseded mapping definitions for their defined retention period; after expiration, obsolete definitions may be deleted while audit retains mapping IDs, versions, and provenance metadata. This is an explicit demo/PoC concession. Production may require archival or retention for as long as historical runs depend on the definitions. The duration remains unresolved and must not be confused with the 24-hour business-record review window (FR-022–FR-025; NFR-7.6).

**Requirement — Run-specific mapping explainability:** Each run must reference the exact mapping ID/version actually used. Its rules must remain retrievable during the mapping-definition retention period so a reviewer can understand how that run mapped the data; storing a duplicate rule snapshot in every run is not required. After an obsolete definition is deleted, audit references remain but rule inspection is no longer guaranteed. This is a demo/PoC concession, distinct from the retention of source/result values (FR-030; NFR-6.6–7.6).

## 5. Workload assumptions

**Requirement — Upload record limit:** Enforce an operator-configured maximum, initially 50 data records per file, excluding the CSV header. Reject the entire oversized upload with a useful explanation before mapping search or AI inference. Do not truncate the file or process only records within the limit. The partial-conversion option applies to eligible processing runs, not to bypassing this intake limit (FR-005).

**Estimate — Typical weekly demo users:** 0–5 people trying the interactive application per week after launch. Erick considers higher use to exceed his current hopes and expectations. This is a planning estimate, not an enforced capacity limit or a maximum traffic forecast.

**Known — User-reported traffic evidence:** Erick reports 176 LinkedIn profile views over the preceding 90 days. This figure has not been independently verified and counts profile views, not necessarily unique people or application users.

**Assumption — Referral conversion:** Erick considers conversion of 10–25% of those profile views into demo-site visits a successful outcome. This is an unvalidated conversion hypothesis, not measured referral behavior.

**Estimate — Implied referral volume:** At the reported profile-view rate, 10–25% conversion implies approximately 18–44 demo-site visits per 90 days, or 1.4–3.4 per week on average (176 × conversion rate × 7/90). This is consistent with the selected 0–5 weekly-user planning range but does not establish actual interactive usage or peak demand.

**Unknown — Visits to active use and bursts:** The fraction of demo-site visitors who submit an upload, traffic from other sources, and short-term arrival clustering remain unmeasured. LinkedIn profile views alone do not validate these factors. Revisit the estimate after launch using available evidence; this does not add a product-analytics implementation requirement.

**Estimate — Uploads per active visit:** Typically one upload; possibly two if the visitor is particularly curious. Source: Erick's estimate in this working session. AI mapping revisions within an upload are separate from additional uploads; linked reprocessing may also add processing runs without a new upload.

**Estimate — Indicative weekly visitor uploads:** Combining 0–5 active visitors with one typical upload suggests 0–5 visitor uploads in a typical week; if each of five visitors uploads twice, that would be 10 uploads. This is scenario arithmetic, not a maximum-capacity requirement, and excludes Erick's own development/testing activity and additional reprocessing runs.

**Requirement — Concurrent visitor capacity:** Target comfortable use by three simultaneous demo visitors; two is the minimum acceptable capacity. Erick regards five concurrent visitors as beyond current expectations, not a committed v1 capacity target. This is a capacity objective rather than a forecast, an enforced admission limit, or a choice of processing concurrency.

**Requirement — Overlapping submissions:** If all three target concurrent visitors submit valid, in-limit uploads at approximately the same time, each should receive its applicable system response within the normal interactive wait target, without a relaxed target solely because the submissions overlap. Source: Erick's selection in this working session. Human review time is separate from system processing time; mapping proposals still require confirmation before transformation. This is an externally observable capacity requirement, not a choice of processing implementation.

**Requirement — Response versus timeout:** NFR-9.1 establishes an interactive experience; NFR-9.2 requires a graceful failed state and no hidden continuing processing after the defined timeout. Normal response targets and the hard failure timeout must be distinguished. Exceeding a normal response target alone does not require failure; reaching the separately defined hard timeout does.

**Requirement — Initial AI proposal response target:** For a valid, in-limit upload requiring a new AI mapping, the normal target is to present the proposal within 10 seconds after upload completion. This includes intake validation, approved-mapping search, and AI proposal preparation, rather than only model inference. Apply the same target under the agreed three-visitor overlapping-submission scenario. Source: Erick's selection in this working session. This is a desired, unvalidated target, not a measured result, a guarantee for every request, or the hard processing timeout.

**Unknown — Response-target validation:** Validate the 10-second proposal target during the vertical slice with representative supported files and three overlapping submissions. File-byte limits, measurement conditions, and the statistical acceptance criterion remain unresolved. The AI revision target, final acceptance criteria for the provisional confirmed-transformation target, and the hard failure timeout remain open.

**Unknown — AI revision response target:** Leave the numeric target unresolved pending representative testing. Erick identified that different types of correction hints may lead to different response behavior and is open to allowing longer than the initial proposal target. Do not assign a longer duration without evidence or automatically apply the initial 10-second target to revisions.

**Assumption — Hint-dependent revision latency:** Correction-hint complexity may affect revision response time; this has not been measured. Validate using representative straightforward and more involved hints under single-visitor and three-visitor overlapping use. Use the results to propose a revision response target and a bounded failure timeout for Erick's review. An unresolved target does not permit unbounded processing.

**Assumption — Provisional confirmed-transformation target:** Aim to complete transformation and make results available within 10 seconds after the user confirms processing, for a valid, in-limit file of up to 50 records. This excludes AI mapping generation/revision and human review time. Erick selected a 10-second target while leaving the final performance commitment unresolved until testing. Evaluate it under both single-visitor use and the agreed three-visitor overlapping-submission workload; do not treat it as measured performance or a hard failure timeout.

**Unknown — Confirmed-transformation validation:** Test representative successful, partially successful, and whole-file-failure runs to establish whether the provisional target is practical and what response-time commitment is supportable. Preserve required storage and audit outcomes when measuring completion.

**Unknown — File sizes and byte limit:** Erick deferred file-size estimates until representative source samples are available after the canonical Company model is defined. The 50-record limit alone does not constrain total bytes, column count, or field length. Measure representative file sizes and variation, then propose an upload-byte limit for review before public launch. No numeric byte limit is selected here.

**Assumption — Sample-based workload validation:** Representative source samples should inform file-size estimates and subsequent performance/cost tests. Canonical-model size alone is insufficient because input files may include extra unmapped fields. Owner: Erick; review point: sample preparation and vertical-slice validation.

## 6. Trust and network boundaries

**Requirement — v1 target-system boundary:** Stored and displayed canonical transformation results in the demo's canonical data store represent successful delivery and the theoretical downstream system of record for v1. v1 does not require delivery to a separate target application. Source: Erick's clarification in this working session.

**Unknown — Production evolution: downstream interface boundary:** A production implementation would introduce an interface to the target company's authoritative application or data stores. Its trust boundary, delivery contract, network path, and failure behavior remain unresolved and depend on the actual enterprise context. No such external target interface is required for v1.

**Requirement — Earlier unsuccessful endpoint:** If the AI attempt/revision allowance is exhausted and the proposal is still incomplete or ambiguous, the workflow ends before confirmed transformation and successful-result storage. Show the unresolved outcome; no further inference or approval of an incomplete mapping is permitted. If the latest proposal is complete and unambiguous, confirmation remains available under FR-015. Preserve required audit/provenance metadata and apply retention/cleanup rules. Source: Erick's clarification; FR-012–FR-018 and NFR-5.3–5.4. This mapping-stage stop must not be confused with a completed transformation run's outcome.

**Requirement — Public human-operated access:** Any person who discovers the public link may access the demo without an invitation from Erick. Interactive use is intended for humans directly operating the demo; bots and scripted/automated consumption should be blocked or constrained. Source: Erick's clarification in this working session. No specific verification, identity, or network mechanism is selected here.

**Assumption — Meaning of direct human use:** Interpret Erick's phrase “in-person users” as a human actively interacting with the application remotely, not a requirement for physical presence with Erick.

**Requirement — Layered abuse and cost protection:** Apply the existing per-workflow, per-user/workspace, and owner-level consumption controls (NFR-8.3–8.5) alongside controls intended to deter automated use. A human-verification check alone must not be treated as proof that subsequent use is non-automated or financially bounded.

**Requirement — Visitor verification experience:** Require a brief human-verification step without account registration before creating a temporary interactive session or accepting cost-generating processing. The specific interaction and implementation remain unresolved. Human verification does not replace workspace isolation, usage limits, or cost controls (FR-047).

**Requirement — Failed verification:** If human verification fails, block interactive processing while keeping the public project explanation and architecture information accessible. Source: Erick's confirmation in this working session. Public architecture information must not expose private workspace data or credentials.

**Requirement — Verification interaction preference:** Prefer a low-friction background check or simple checkbox-style interaction over account registration or a visual puzzle. This describes the desired visitor experience and does not select a vendor or implementation.

**Unknown — Human-verification acceptance criteria:** The specific interaction, acceptable verification delay, retry behavior after a failed check, and validation criteria for deterring automated usage remain unresolved. Complete exclusion of bots is not an established capability or guarantee.

**Requirement — Existing data trust boundaries:** Enforce logical isolation of workspace source files, results, rejected records, and runs. Shared mappings exclude private source values, prompts, user identity, and audit history. AI receives only the information needed for its task; actual AI context is not retained after the interaction. Persisted data and data in transit are encrypted, and operational logs exclude business-record values (FR-020; NFR-3.1–3.8). These requirements apply despite synthetic-data guidance.

**Unknown — Detailed network flows:** The logical interactions are visitor uploads/review/results, application access to private run state and shared mappings, and AI invocation where needed. Hosting boundaries, external AI/verification providers, connection characteristics, and permitted outbound destinations remain design decisions. No separate downstream enterprise endpoint is required for v1.

## 7. Failure and recovery characteristics

**Requirement — Approved-mapping reuse during AI outage:** Users should still be able to process files using compatible, previously approved mappings when AI is temporarily unavailable. Existing human confirmation, canonical-version compatibility, deterministic transformation, and audit requirements continue to apply. Source: Erick's selection in this working session. This desired availability behavior is subject to the unresolved compatibility-evaluation dependency below; no implementation is selected here.

**Assumption — AI-independent reuse path:** Achieving this behavior requires the critical reuse path, including determining mapping compatibility with the submitted source, to remain usable without the unavailable AI capability. Searching approved mappings before generating a new AI proposal does not itself establish that compatibility evaluation is AI-independent. Erick explicitly identified this dependency.

**Unknown — Compatibility evaluation dependency:** Determine whether existing-mapping matching relies on AI. If it does, document that the AI-outage availability objective cannot be met by that path without an alternative; revisit the tradeoff explicitly rather than claiming reuse remains available. Evaluate this during architecture selection and validate the selected behavior with an AI-unavailable scenario in the vertical slice.

**Assumption — Production evolution: partial availability:** Preserve the objective of processing with valid approved mappings during an AI outage in enterprise/production evaluation. The same compatibility-evaluation dependency applies; this is not an assumption that the entire application can operate without AI.

**Requirement — Existing failure/recovery baseline:** Failed processing is visible and does not automatically retry. A user-initiated retry creates a new linked run without changing the original outcome. The defined processing timeout stops hidden continuing work, and partial-conversion settings govern whether valid records may be stored when other records fail (NFR-4.1–4.5, NFR-9.2; FR-026–FR-034).

**Requirement — AI outage without an established compatible mapping:** If AI is unavailable and no compatible approved mapping can be established, explain the limitation and stop the affected workflow. The user may try again later; v1 does not automatically retry or resume the request when AI recovers. A later attempt remains subject to workspace availability, data retention, and usage limits; this does not promise cross-visit recovery. Source: Erick's confirmation in this working session.

**Requirement — Established recovery objectives:** There is no formal v1 uptime SLA. Service recovery has a best-effort 24-hour RTO, potentially longer when the owner is unavailable. Approved mappings and audit/provenance initially target a one-hour RPO, subject to cost/complexity validation. Transient source/result/rejected data and in-progress state have no formal RPO; loss may require a new user-initiated run (NFR-9.3–9.5). Continuing work after a browser closes does not promise survival of every infrastructure failure.

## 8. Architecture-driving quality attributes

**Requirement — Leading demonstration quality:** Prioritize correct, explainable results when assessing how convincingly v1 demonstrates its value. Source: Erick's selection in this working session. This prioritization does not make mandatory security, human control, cost protection, retention, recovery, or performance requirements optional.

**Requirement — Evidence of correctness and explainability:** Use the existing requirements as the basis: complete and unambiguous mappings; no fabricated missing source values; visible mapping rules, sample results, and unmapped fields; human confirmation; deterministic transformation; clear rejection reasons; and mapping/canonical-version provenance linked to run outcomes (FR-008–FR-018, FR-024–FR-041; NFR-5.1–5.3, NFR-6.3–6.6). Human approval and a successful processing status alone do not prove semantic correctness.

**Unknown — Representative correctness evidence:** Specific expected field values and validation examples depend on the canonical Company model and representative source samples. Define and validate those examples during model/sample preparation and the vertical slice rather than inventing them in this profile.

**Requirement — Cost, operability, and observability baseline:** Keep idle cost deliberately low, enforce bounded consumption, and stop cost-generating interactive use at the owner-level financial threshold until manual intervention. Maintain append-only audit metadata, operational counts, latency, failures, AI usage, storage/quota metrics, and actionable failure/cost alerts. Deployment and teardown must be repeatable and documented (NFR-6.1–6.8, NFR-8.1–8.5, NFR-10.1–10.3). Exact budgets and implementation mechanisms remain unresolved; none is invented by this profile.

## 9. Assumptions and unknowns

**Assumption — Highest-priority feasibility hypothesis:** AI can propose useful source-to-canonical mappings and incorporate valid user correction hints to produce correct, complete, unambiguous mappings for representative supported inputs within the bounded workflow. This is not yet demonstrated. Source: Erick identified AI usefulness as the core concept and first validation priority.

**Known — Consequence if the hypothesis fails:** Erick considers failure of this capability a threat to the central AI-assisted demo premise. Deterministic processing and mapping reuse would still have technical value, but would not by themselves establish the intended AI-assisted onboarding demonstration.

**Requirement — First validation priority:** Test AI proposal and human-guided revision usefulness before investing heavily in the full build. Use the eventual canonical Company model, representative synthetic inputs, and expected results to judge correctness and completeness rather than merely counting populated mappings. Apply the existing one-initial-attempt/two-revision allowance and no-fabricated-data requirement. Include inputs where missing source information should lead to an explained inability to complete a mapping rather than invented values. Owner: Erick; review point: initial feasibility test/vertical slice.

**Requirement — Initial feasibility acceptance boundary:** Before investing heavily in the full build, demonstrate repeatable success across several meaningfully different representative source examples, including a successful user-guided correction. Source: Erick selected this standard in this working session. Judge success against expected canonical results, mapping completeness, and existing guardrails within the permitted AI attempts. A single curated successful example is insufficient; no numeric success-rate threshold is committed at this stage.

**Unknown — Feasibility test details:** Exact examples, number of repeated trials, and treatment of inconsistent results remain to be specified once the canonical model and representative inputs exist. Record outcomes and limitations without claiming measured reliability or retaining prohibited AI context in the application. Known synthetic test fixtures and expected results may define test cases; application AI-interaction retention remains governed by NFR-3.5.

Other unresolved items remain labeled in their relevant sections, including model definition, matching dependencies, performance targets, file sizes, consumption limits, and production policies. Source-verification items are tracked above.

## 10. Architecture questions intentionally left unresolved

Architecture and service selection remain outside this profile.

- Does mapping compatibility evaluation require AI, and how does that affect the objective of approved-mapping reuse during AI outages? No matching approach has been selected. This question must be resolved before the availability claim can be validated.
- What canonical Company fields, identity semantics, validation rules, and bounded transformation operations define the target contract? Detailed definition remains with the planned model/design work; repeated-company handling is provisional until then.
- What minimum source information and correction context does each AI operation need, and how will proposals be checked for completeness and correctness? Respect the existing minimum-exposure and human-confirmation requirements.
- How will the bounded preview and confirmed processing paths satisfy partial-conversion rules, duplicate-request protection, visitor departure behavior, and bounded timeouts while preserving their established separation?
- How will temporary workspace access, human verification, shared mapping reuse, private provenance, and data deletion meet the established trust boundaries without enterprise user administration?
- Which implementation can meet the workload and response targets within a justified cost ceiling? Numeric quotas, storage cap, hard timeout, mapping retention, and cost thresholds remain unresolved pending design and measurement.

These are inputs to later architecture comparison and ADRs, not selected solutions. The AI feasibility test precedes substantial build investment; this profile does not prescribe its model, services, or implementation.
