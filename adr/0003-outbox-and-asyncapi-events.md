# ADR 0003 - Outbox and AsyncAPI Events

## Status

Accepted

## Context

DEVJUR Platform needs to publish integration events when tenant and platform state changes. Event publication must not break transactional consistency between persisted business data and messages sent to Kafka.

## Decision

The API writes integration events to the tenant-local `outbox_event` table in the same transaction as the related business change.

A publisher process reads pending outbox rows and publishes them to Kafka.

Event schemas, message names and channel names are versioned in `devjur-contracts` using AsyncAPI before the events are exposed outside the producing service.

## Consequences

- Business transactions can commit even when Kafka is unavailable.
- Outbox rows remain pending until a publisher can deliver them.
- AsyncAPI contracts are required before introducing external integration events.
- Consumers must be idempotent because delivery can be retried.
- Retry policies and dead-letter handling are later hardening work.
