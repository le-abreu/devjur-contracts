# ADR 0002 - Keycloak-First Authorization

## Status

Accepted

## Context

DEVJUR Platform needs tenant-aware authorization that stays consistent across backend APIs, frontend navigation and infrastructure-managed identity resources.

## Decision

Keycloak is the primary source for roles, groups, clients and scopes.

The API validates token issuer, tenant and roles, and exposes a navigation permission projection for frontend clients.

The API still enforces tenant, data and domain invariants. Authorization from Keycloak does not replace backend checks that protect tenant boundaries or business rules.

## Consequences

- Realm, client, role, group and scope changes must be managed as identity configuration.
- Frontend navigation must be derived from the API permission projection.
- Backend services must reject tokens with invalid issuer, tenant or role claims.
- Backend services remain responsible for tenant isolation, data integrity and domain invariants.
