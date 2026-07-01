# Contracts Module Model

Phase 5 starts the first business module with a deliberately small contract model. The goal is to validate tenant isolation, Keycloak-first permissions, audited storage, OpenAPI-driven frontend integration and later domain events without implementing full legal workflow complexity in the first cut.

## Scope

The minimum module supports:

- Creating tenant-local contract drafts.
- Listing and filtering contracts by status or search text.
- Reading a contract detail.
- Updating editable draft metadata.
- Linking files uploaded through the platform storage API as contract attachments.

The first cut does not implement clause libraries, approval workflows, signing integrations, obligation tracking, entity master data, full document versioning or external integrations.

## Aggregate

`Contract` is the tenant-local aggregate root.

Required fields:

- `contractId`: immutable UUID.
- `tenantKey`: resolved from the request host and authenticated token.
- `title`: human-readable contract name.
- `type`: one of `SERVICE_AGREEMENT`, `PURCHASE_AGREEMENT`, `NDA`, `ADDENDUM`, `OTHER`.
- `status`: lifecycle state.
- `parties`: embedded party snapshots for the first cut.
- `createdAt`, `updatedAt`, `createdBy`.

Optional fields:

- `contractNumber`: tenant-unique business identifier.
- `summary`: short text description.
- `amount` and `currency`.
- `effectiveDate` and `expirationDate`.
- `updatedBy`.

## Parties

Parties are snapshots inside the contract boundary until the entity master module exists.

Each party has:

- `partyId`
- `role`: `OWNER`, `COUNTERPARTY`, `GUARANTOR`, `WITNESS`
- `displayName`
- optional `document`
- optional `email`
- `primary`

At least one party is required. The primary counterparty is used in list projections.

## Attachments

Attachments reference files already uploaded through `/api/platform/files`.

Each attachment has:

- `attachmentId`
- `contractId`
- `fileId` and resolved platform file metadata
- `attachmentType`: `DRAFT`, `SIGNED`, `SUPPORTING_DOCUMENT`, `OTHER`
- optional `description`
- `createdAt`, `createdBy`

The contract module does not own binary storage. It owns the relationship between the contract and platform file metadata.

## Statuses

The initial lifecycle is:

- `DRAFT`: editable metadata and attachments.
- `IN_REVIEW`: ready for legal or business review.
- `APPROVED`: approved internally.
- `REJECTED`: review rejected.
- `EXECUTED`: signed or otherwise effective.
- `CANCELLED`: closed without execution.

The OpenAPI contract allows status updates through `PATCH /api/contracts/{contractId}` for the first cut. Backend implementation may enforce stricter transition rules later without changing the response model.

## Permissions

All endpoints are tenant-scoped and authenticated with bearer JWT.

The planned role mapping for backend implementation is:

- Read operations: `DEVJUR_USER`.
- Create, update and attach file: `DEVJUR_ADMIN` for the first cut.

More granular roles can be introduced later as Keycloak module/action roles.

## API Surface

The OpenAPI source of truth is:

```text
openapi/devjur-platform-api.yaml
```

Initial paths:

- `GET /api/contracts`
- `POST /api/contracts`
- `GET /api/contracts/{contractId}`
- `PATCH /api/contracts/{contractId}`
- `GET /api/contracts/{contractId}/attachments`
- `POST /api/contracts/{contractId}/attachments`
