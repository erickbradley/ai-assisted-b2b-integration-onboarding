# AI-Assisted B2B Integration Onboarding Platform

An AWS-based reference architecture for accelerating B2B integration onboarding through AI-assisted schema mapping, human-in-the-loop validation, and reusable integration mappings.

## Status

Architecture and MVP design in progress.

## Design documents

- [Problem definition](docs/problem-definition.md)
- [v1 functional requirements](docs/functional-requirements-v1.md)
- [v1 nonfunctional requirements](docs/nonfunctional-requirements-v1.md)
- [Pre-build application profile](docs/application-profile-pre-build.md)


## Current demonstration scenario

Support ticket CSV -> human-reviewed mapping -> deterministic CRM-shaped Support Case results stored inside the demo. No live CRM connector is required.

- [System context](docs/system-context.md)
- [Architecture design drivers](docs/architecture-design-drivers.md)
- [Support Case contract](docs/support-case-contract-v1.md)
- [Target JSON Schema](schemas/support-case-v1.schema.json)
- [Synthetic examples and expected results](examples/support-cases/README.md)
- [Architecture workflows](docs/architecture-workflows.md)
- [Logical architecture and responsibilities](docs/logical-architecture.md)
