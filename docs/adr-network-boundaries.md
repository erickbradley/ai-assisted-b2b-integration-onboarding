# ADR: Network boundaries and anonymous demo access

Status: Accepted design direction; implementation and validation pending.
Decision date: 2026-11-17

## Context

The public demo accepts small, synthetic support-ticket CSV files and produces human-approved CRM-shaped results. Uploaded files still require cautious handling because safeguards can miss prohibited content (NFR-2.1). Intermittent usage, limited maintenance capacity and the provisional $20/month operating ceiling constrain additional infrastructure. See the [requirements](nonfunctional-requirements-v1.md), [orchestration ADR](architecture-orchestration.md) and [service matrix](aws-service-mapping-draft.md).

Erick initially expected every AWS resource to reside in a VPC subnet. The architecture review separated network placement, invocation authorization and data access. A customer VPC is not required merely to make S3, DynamoDB or Lambda access restricted. It is justified here by a deliberate outbound-connectivity policy for selected workers, not by treating all managed services as subnet resources.

## Decisions

### 1. Anonymous demo entry

Keep access lightweight: no user accounts or passwords. Static page assets may load before verification, but the interface presents human verification before enabling protected demo functionality. The backend validates Turnstile evidence before granting a new verified session. A valid verified session can be reused, including entry through a direct or bookmarked URL. Missing or expired verified sessions require verification. Every protected backend operation enforces current workspace/session access and applicable usage limits; a browser-only gate is insufficient.

Human verification establishes demo eligibility, not personal identity. It reduces automation without guaranteeing a human or preventing abuse by a verified session. Session expiry, credential handling and quota values remain open; they are separate from data-retention and human-review timeouts. The Turnstile token is exchanged for application session access, not retained as a reusable session credential.

### 2. Restricted workers

Apply a common policy to the transformation executor, mapping-search/evaluation Lambda and cleanup Lambda: required AWS connectivity, with no general outbound internet route. Use a customer VPC and private-subnet connectivity for the Lambda-based workers. The executor's runtime/engine is still open; this is a network requirement on its eventual implementation, not a new selection of Lambda over Glue.

For the Lambda candidate, use S3 and DynamoDB gateway endpoints and restrict routes, execution roles and endpoint/resource policies to required access. S3 and DynamoDB remain regional services outside the VPC. Do not add NAT or public subnets merely to access those services. Validate every actual SDK call, extension, dependency and operational integration before claiming a complete connectivity design. Package ordinary libraries at build/deployment time where practical; a library dependency alone does not imply runtime internet access.

Private subnet placement alone does not enforce this policy. Check IPv4 and IPv6 routes, security-group egress, network permissions and any newly introduced connectivity. VPC connectivity does not replace Lambda invocation permissions. This is a restriction on general outbound application connectivity, not a claim of total isolation or comprehensive prevention of data exfiltration; authorized services and DNS remain separate controls to assess.

### 3. Application-backend exception

Keep the entry/application backend on default Lambda networking without a customer-VPC connection for v1. Its known external dependency is Cloudflare Siteverify. Use a fixed verification destination in code, server-side secret handling, narrowly scoped AWS permissions, input validation and consumption controls. Do not accept a visitor-supplied verification URL.

Explicit residual risk: these application controls do not prevent compromised code from connecting to arbitrary external destinations. Network-enforced Cloudflare-only filtering is deferred. A NAT gateway alone would not supply destination filtering. A subnet/route design plus proxy or firewall would add cost, maintenance and another failure boundary.

The current backend also owns intake, approval, polling and file delivery. The exception must not be described as affecting only a token-only function while deploying all those responsibilities together. Physical Lambda grouping remains open; scope each deployed role to its actual operations and reassess the residual risk for any function handling business data. No new standalone verification microservice is selected.

### 4. Diagram semantics

Distinguish public entry points, restricted service access and actual network containment. CloudFront and Route 53 are outside customer subnets. API Gateway, Step Functions, Bedrock, S3, DynamoDB and other managed regional services are outside the customer VPC. CloudFront accesses the private frontend S3 origin through OAC; business-data buckets are not public website origins.

Show the customer VPC and private-subnet connectivity for restricted workers. For Lambda, this represents network interfaces/connectivity; execution remains in the Lambda-managed service environment. Do not draw S3 or DynamoDB inside a private subnet. Step Functions/Scheduler invoke workers through authorized service APIs, not through an internet-facing listener on the worker.

The network page is a first-draft allocation. Region, subnet count, Availability Zones, CIDRs and detailed routes remain open. It depicts one aggregate private-subnet group, not a selected single-AZ deployment. Review a multi-AZ layout before implementation. Other pages show logical dependencies rather than physical subnet placement.

## Alternatives and consequences

| Alternative | Benefit | Why selected or deferred |
| --- | --- | --- |
| Default networking for all Lambda functions | Least network setup; managed-service access still protected by permissions | Not selected for restricted workers because it leaves general outbound internet connectivity available |
| VPC-connect restricted workers; retain backend exception | Adds an independently enforced outbound boundary where practical while keeping external verification simple | Selected direction; requires additional IaC, permissions and connectivity tests |
| Put every backend behind selective internet egress | More consistent network-level destination control | Deferred for v1 because filtering, routing, resilience and ongoing cost are disproportionate to the current demo constraints |
| Place frontend/data services in public/private subnet boxes | Familiar visual tier separation | Rejected because it misrepresents the selected managed services; access restrictions do not imply subnet placement |

S3/DynamoDB gateway endpoints have no additional hourly or processing charge. This does not make the complete architecture free. New interface endpoints, filtering components, logs and other resources may add costs. As an illustration only, AWS's Ohio NAT example charges $0.045/hour: about $32.85 for 730 hours before processing and other charges, exceeding the current provisional monthly ceiling. No NAT gateway is selected in this ADR.

This is enterprise-informed proportional control: document the exception and residual risk rather than claim uniform isolation. Revisit the exception for real customer-data use, stricter contractual/security obligations, new outbound dependencies, broader backend privileges or a funded production deployment. New connectivity requires an explicit review; do not silently add internet routes to make a failing task work.

## Validation before implementation sign-off

- Confirm executor engine and all runtime dependencies; preserve the no-general-internet policy or reopen the decision explicitly.
- Inventory mapping evaluation and cleanup calls, including any AI, direct notification, metrics, secrets or callback API usage. If a restricted worker needs another AWS API, evaluate its private endpoint and cost rather than grant general egress automatically.
- Verify legitimate S3/DynamoDB operations succeed and an unrelated external HTTPS request fails from each restricted worker. Check both address families and routes. Verify invocation authorization independently.
- Prove session/usage checks reject direct unauthenticated API requests; confirm bookmarked entry reuses only valid verified sessions.
- Validate private S3 policies, OAC distribution binding, endpoint policies and workspace isolation; network access alone does not authorize data access.
- Estimate idle, expected and higher-use costs. Design concrete subnet/AZ layout, operational connectivity and least-privilege roles in IaC.
- Carry forward the orchestration ADR's open issues, particularly inference-context retention, publication/audit consistency, duplicate claims and retention versus active execution. Network changes do not resolve those issues.

## References

- [Editable diagram, page 4](../assets/diagrams/aws-architecture.drawio)
- [Lambda VPC connectivity](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html)
- [Lambda internet connectivity](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc-internet.html)
- [Lambda governance and network controls](https://aws.amazon.com/blogs/compute/building-aws-lambda-governance-and-guardrails/)
- [Gateway endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html)
- [Security-group rule destinations](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html)
- [AWS Network Firewall domain filtering](https://docs.aws.amazon.com/network-firewall/latest/developerguide/stateful-rule-groups-domain-names.html)
- [VPC pricing](https://aws.amazon.com/vpc/pricing/)
- [Turnstile server validation](https://developers.cloudflare.com/turnstile/get-started/server-side-validation/)

Erick selected these policies during the architecture ownership walkthrough. They are accepted design choices with explicit feasibility work, not a claim of deployed security or production compliance.
