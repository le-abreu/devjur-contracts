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

## Local Validation

Run the local stack with the tenant host names. Use `demo.localhost` for the web
application and `demo.api.localhost` through the API `Host` header.

Infrastructure:

```bash
cd devjur-platform-infra
docker compose -f docker-compose.yml up -d
```

If local ports `9000/9001` or `3000` are already used, start the infra with the
versioned local ports override. In that case MinIO is available at
`http://localhost:9100`.

API for the full foundation-v4 flow:

```bash
cd devjur-platform-api
mvn spring-boot:run \
  -Dspring-boot.run.arguments="--devjur.storage.enabled=true --devjur.storage.endpoint=http://localhost:9100 --devjur.mail.enabled=true --devjur.features.openfeature.enabled=true --devjur.observability.otel.enabled=true"
```

Web:

```bash
cd devjur-platform-web
npm start -- --host 0.0.0.0 --port 4200
```

Open:

```text
http://demo.localhost:4200/platform/contracts
```

Demo credentials:

- Username: `admin.demo`
- Password: `Admin@12345`

Feature flag requirement:

- `platform.contracts.enabled=true` in local Unleash.

Manual UI checklist:

1. Log in with the demo user.
2. Open `Contratos`.
3. Create or select a contract.
4. Select a local file in the contract detail.
5. Click `Enviar e anexar`.
6. Confirm the file appears under `Arquivos anexados`.
7. Click `Baixar` and confirm the downloaded content matches the uploaded file.

Backend smoke validation:

```bash
TOKEN=$(curl -sS -X POST http://localhost:8081/realms/devjur-demo/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "grant_type=password&client_id=devjur-platform-web&username=admin.demo&password=Admin%4012345" \
  | python3 -c 'import json,sys; print(json.load(sys.stdin)["access_token"])')

curl -sS http://localhost:8080/api/platform/features/platform.contracts.enabled/evaluate \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN"
```

Expected feature response:

```json
{
  "flagKey": "platform.contracts.enabled",
  "enabled": true,
  "variant": null,
  "reason": "UNLEASH"
}
```

Validation executed on 2026-07-01:

- API health returned `UP`.
- `platform.contracts.enabled` returned `enabled=true` with reason `UNLEASH`.
- A contract was created from the UI at `http://demo.localhost:4200/platform/contracts`.
- A text file was uploaded through the contract detail screen.
- The uploaded file appeared in `Arquivos anexados`.
- The `Baixar` action was available and completed for the attached file.
- API smoke test created contract `CTR-E2E-20260701140707`, uploaded file
  `contrato-e2e-20260701140707.txt`, attached it, downloaded it and matched
  the downloaded bytes with the uploaded content.
