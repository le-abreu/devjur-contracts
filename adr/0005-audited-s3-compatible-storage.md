# ADR 0005 - Audited S3-Compatible Storage

## Status

Accepted

## Context

DEVJUR Platform needs file upload and download support for future document and contract modules. Files are tenant data, and access to them must be auditable from the first storage foundation.

## Decision

MinIO is the local S3-compatible storage service.

The API mediates upload and download in Foundation V4 so metadata and audit records can be written in the tenant database for every file operation.

The tenant database stores file metadata and audit records. The object bytes stay in S3-compatible storage.

Object keys include the tenant key and file id.

Direct browser-to-S3 presigned uploads are deferred until audit callbacks, validation policies and security controls are designed.

## Consequences

- Upload and download are slower than direct-to-storage flows but auditable immediately.
- Metadata queries remain tenant-local and do not require listing storage buckets.
- Future production storage can use AWS S3 or another compatible service.
- Presigned upload support requires a later ADR or amendment that preserves audit guarantees.
