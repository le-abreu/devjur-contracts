# ADR 0001 - Contracts Repository as Source of Truth

## Status

Accepted

## Context

DEVJUR Platform uses separated repositories for backend, frontend, infrastructure and shared contracts. Without a central contracts repository, API and event schemas would drift across teams and repositories.

## Decision

The `devjur-contracts` repository is the source of truth for OpenAPI, AsyncAPI, shared schemas and cross-repository ADRs.

## Consequences

- Backend implementations must match OpenAPI contracts.
- Frontend clients must be generated from OpenAPI contracts.
- Kafka events introduced in later phases must be described in AsyncAPI.
- Breaking changes require explicit contract versioning.
