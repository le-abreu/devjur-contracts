# DEVJUR Contracts

Source of truth for DEVJUR Platform public contracts.

## Contents

- OpenAPI specifications for REST APIs.
- AsyncAPI specifications for event contracts.
- Shared JSON Schemas for event payloads and envelopes.
- ADRs for architectural decisions shared across repositories.

## Foundation API

The phase 1 OpenAPI contract is available at:

```text
openapi/devjur-platform-api.yaml
```

## Phase 2 Contracts

The platform provisioning and navigation permission contracts are defined in the same OpenAPI document.

ADR 0002 documents the Keycloak-first authorization decision:

```text
adr/0002-keycloak-first-authorization.md
```

## Event Contracts

Platform integration events are defined with AsyncAPI and shared JSON Schemas:

```text
asyncapi/devjur-platform-events.yaml
schemas/events/common-event-envelope.schema.json
schemas/events/tenant-provisioned.schema.json
```

ADR 0003 documents the outbox and AsyncAPI event versioning decision:

```text
adr/0003-outbox-and-asyncapi-events.md
```
