# ADR 0004 - Enterprise Auxiliary Services

## Status

Accepted

## Context

DEVJUR Platform needs feature flags, UI translation, email capture and observability as platform capabilities before the first business module. These capabilities must be available locally without making unit tests depend on external services.

## Decision

Unleash is the local feature flag service.

The backend evaluates flags behind an OpenFeature-oriented abstraction and must be disabled-safe by default.

Tolgee is the UI i18n service for labels, messages, validation text and tooltips.

Mailpit is the local SMTP capture service.

LGTM and OpenTelemetry Collector are the local observability foundation.

Auxiliary service adapters must keep stable application-facing interfaces so implementations can be replaced without leaking vendor APIs into business modules.

## Consequences

- Local development can validate enterprise platform integrations before business modules depend on them.
- Unit tests can run without Unleash, Tolgee, Mailpit, MinIO or LGTM.
- Feature flag and i18n failures must degrade predictably instead of blocking login or tenant resolution.
- Production service choices can evolve behind the same ports.
