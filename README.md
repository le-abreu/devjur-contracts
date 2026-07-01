# DEVJUR Contracts

Source of truth for DEVJUR Platform public contracts.

## Contents

- OpenAPI specifications for REST APIs.
- AsyncAPI specifications for event contracts.
- Shared JSON Schemas for event payloads and envelopes.
- ADRs for architectural decisions shared across repositories.
- Operational documentation for local development and validation.

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

## Phase 4 Contracts

Enterprise auxiliary service contracts are defined in the OpenAPI document:

```text
openapi/devjur-platform-api.yaml
```

Phase 4 adds contracts for:

- Feature flag evaluation.
- Audited file upload, metadata lookup and download.
- Tenant audit lookup.
- Controlled platform test email dispatch.

ADR 0004 documents the auxiliary services decision:

```text
adr/0004-enterprise-auxiliary-services.md
```

ADR 0005 documents audited S3-compatible storage:

```text
adr/0005-audited-s3-compatible-storage.md
```

## Phase 5 Contracts Module

The first business module contract and minimum domain model are documented at:

```text
openapi/devjur-platform-api.yaml
docs/contracts-module-model.md
```

## Local Development

The foundation-v4 local workflow, service URLs, credentials, smoke tests and shutdown commands are documented at:

```text
docs/local-development.md
```
