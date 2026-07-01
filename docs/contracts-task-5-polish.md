# Contracts Module Task 5 Polish

Task 5 closes the first usability polish pass for the Contracts module after the
minimum backend, frontend and local end-to-end validation are merged to `main`.

## Scope

- Add direct file upload from the contract detail screen.
- Attach the uploaded file to the selected contract without requiring manual `fileId` entry.
- Show attached files in the contract detail screen.
- Allow downloading attached files from the contract detail screen.
- Document the local end-to-end validation flow for the Contracts module.
- Document the local feature flag requirement for `platform.contracts.enabled`.

## Repositories

- `devjur-platform-web`: contract detail upload, attachment list and download UX.
- `devjur-platform-api`: backend adjustments only if the frontend polish exposes an API gap.
- `devjur-platform-infra`: local feature flag/bootstrap or local validation docs, if needed.
- `devjur-contracts`: OpenAPI/docs updates and E2E validation documentation.

## Acceptance Criteria

- A user can open `http://demo.localhost:4200/platform/contracts`, select a contract,
  upload a file and see it attached to that contract.
- A user can download an attached contract file from the same detail screen.
- The flow keeps using tenant host resolution and bearer authentication.
- The flow works with `platform.contracts.enabled=true` in local Unleash.
- Automated frontend tests cover upload, attach list refresh and download action.
- API tests remain green after any backend adjustment.
- Documentation includes repeatable local validation commands.
