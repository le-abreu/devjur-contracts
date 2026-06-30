# DEVJUR Platform Local Development

Guia operacional para subir, validar e parar a DEVJUR Platform localmente.

## Repositorios

Execute os comandos a partir da raiz do workspace:

```bash
cd /home/leandroabreuferreira/Develop/workspace/workspace_devjur
```

Repositorios locais:

- `devjur-contracts`: OpenAPI, ADRs e contratos compartilhados.
- `devjur-platform-infra`: Docker Compose, Postgres, Keycloak, Vault, Redis, Kafka, MinIO, Mailpit, Unleash, Tolgee e observabilidade.
- `devjur-platform-api`: API Spring Boot.
- `devjur-platform-web`: frontend Angular.

## URLs

- Web: `http://demo.localhost:4200/`
- API: `http://localhost:8080`
- API tenant demo: `http://demo.api.localhost:8080`
- Keycloak: `http://localhost:8081`
- Vault: `http://localhost:8200`
- Kafka UI: `http://localhost:8082`
- Kafka bootstrap local: `localhost:9092`
- Redis: `localhost:6379`
- MinIO API: `http://localhost:9000`
- MinIO Console: `http://localhost:9001`
- Mailpit SMTP: `localhost:1025`
- Mailpit UI: `http://localhost:8025`
- Unleash: `http://localhost:4242`
- Tolgee: `http://localhost:8083`
- LGTM/Grafana: `http://localhost:3000`
- OTLP gRPC: `localhost:4317`
- OTLP HTTP: `http://localhost:4318`

## Credenciais

Usuario demo:

```text
admin.demo
Admin@12345
```

Keycloak admin local:

```text
admin
admin
```

Vault local:

```text
devjur-root
```

MinIO local:

```text
devjur-minio
devjur-minio-secret
```

Tolgee local:

```text
admin
admin
```

LGTM/Grafana local:

```text
admin
admin
```

Unleash client token local:

```text
default:development.devjur-unleash-client-token
```

## Subir Ambiente

### 1. Infraestrutura

```bash
docker compose -f devjur-platform-infra/docker-compose.yml up -d
docker compose -f devjur-platform-infra/docker-compose.yml ps
```

Servicos esperados:

- `devjur-postgres-control`: `localhost:5433`
- `devjur-postgres-tenant-demo`: `localhost:5434`
- `devjur-postgres-keycloak`: `localhost:5435`
- `devjur-keycloak`: `localhost:8081`
- `devjur-vault`: `localhost:8200`
- `devjur-redis`: `localhost:6379`
- `devjur-kafka`: `localhost:9092`
- `devjur-kafka-ui`: `localhost:8082`
- `devjur-minio`: `localhost:9000`, console `localhost:9001`
- `devjur-mailpit`: SMTP `localhost:1025`, UI `localhost:8025`
- `devjur-unleash`: `localhost:4242`
- `devjur-tolgee`: `localhost:8083`
- `devjur-lgtm`: Grafana `localhost:3000`
- `devjur-otel-collector`: OTLP `localhost:4317` e `localhost:4318`

Se outra stack local ja estiver usando `9000/9001` ou `3000`, crie um override local a partir do exemplo versionado:

```bash
cp devjur-platform-infra/docker-compose.local-ports.yml.example \
  devjur-platform-infra/docker-compose.local-ports.yml
```

Suba com:

```bash
docker compose \
  -f devjur-platform-infra/docker-compose.yml \
  -f devjur-platform-infra/docker-compose.local-ports.yml \
  up -d
```

Com esse override, use MinIO em `http://localhost:9100`, console em `http://localhost:9101` e LGTM/Grafana em `http://localhost:3010`.

### 2. API

```bash
cd devjur-platform-api
mvn spring-boot:run
```

Para validar publicacao de eventos e cache Redis localmente:

```bash
cd devjur-platform-api
mvn spring-boot:run -Dspring-boot.run.arguments="--devjur.events.publishing.enabled=true --devjur.cache.redis.enabled=true"
```

Para validar a foundation-v4 completa:

```bash
cd devjur-platform-api
mvn spring-boot:run -Dspring-boot.run.arguments="--devjur.storage.enabled=true --devjur.mail.enabled=true --devjur.features.openfeature.enabled=true --devjur.observability.otel.enabled=true"
```

Se estiver usando o override de portas do MinIO:

```bash
cd devjur-platform-api
mvn spring-boot:run -Dspring-boot.run.arguments="--devjur.storage.enabled=true --devjur.storage.endpoint=http://localhost:9100 --devjur.mail.enabled=true --devjur.features.openfeature.enabled=true --devjur.observability.otel.enabled=true"
```

Em outro terminal, valide:

```bash
curl -sS http://localhost:8080/actuator/health
```

### 3. Web

```bash
cd devjur-platform-web
npm start -- --host 0.0.0.0 --port 4200
```

Abra:

```text
http://demo.localhost:4200/
```

Use sempre `demo.localhost`, nao `localhost`, porque a resolucao de tenant e feita pelo host.

## Validacoes Rapidas

### Token Keycloak

```bash
TOKEN=$(curl -sS -X POST http://localhost:8081/realms/devjur-demo/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "grant_type=password&client_id=devjur-platform-web&username=admin.demo&password=Admin%4012345" \
  | jq -r .access_token)
```

Sem `jq`:

```bash
curl -sS -X POST http://localhost:8081/realms/devjur-demo/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "grant_type=password&client_id=devjur-platform-web&username=admin.demo&password=Admin%4012345" \
  -o /tmp/devjur-token.json

TOKEN=$(python3 - <<'PY'
import json
print(json.load(open('/tmp/devjur-token.json'))['access_token'])
PY
)
```

### Tenant Atual

```bash
curl -sS http://localhost:8080/api/foundation/tenant/current \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN"
```

### Usuario Atual

```bash
curl -sS http://localhost:8080/api/foundation/me \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN"
```

### Navegacao Por Permissoes

```bash
curl -sS http://localhost:8080/api/platform/permissions/navigation \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN"
```

Resposta esperada para `admin.demo`:

```json
{
  "tenantKey": "demo",
  "entries": [
    {
      "id": "dashboard",
      "label": "Dashboard",
      "route": "/",
      "requiredRole": "DEVJUR_USER"
    },
    {
      "id": "platform-tenants",
      "label": "Tenants",
      "route": "/platform/tenants",
      "requiredRole": "DEVJUR_ADMIN"
    }
  ]
}
```

### Provisionar Tenant Local

```bash
curl -sS -X POST http://localhost:8080/api/platform/tenants \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "tenantKey": "acme",
    "displayName": "ACME Juridico",
    "apiHost": "acme.api.localhost",
    "webHost": "acme.localhost",
    "realm": "devjur-acme",
    "adminUsername": "admin.acme",
    "adminEmail": "admin.acme@devjur.local"
  }'
```

Resposta esperada:

```json
{
  "tenantKey": "acme",
  "displayName": "ACME Juridico",
  "apiHost": "acme.api.localhost",
  "webHost": "acme.localhost",
  "realm": "devjur-acme",
  "status": "ACTIVE",
  "vaultPath": "secret/data/devjur/tenants/acme/database"
}
```

Repetir o mesmo provisionamento deve retornar HTTP `409`.

### Feature Flags

```bash
curl -sS http://localhost:8080/api/platform/features/foundation.v4.files/evaluate \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN"
```

Sem flag cadastrada no Unleash local, a resposta esperada e `enabled=false` com `reason=DEFAULT`.

### Upload/Download De Arquivos

```bash
printf 'DEVJUR foundation-v4 local validation file.\n' > /tmp/devjur-upload.txt

curl -sS -X POST http://localhost:8080/api/platform/files \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@/tmp/devjur-upload.txt;type=text/plain" \
  -o /tmp/devjur-upload-response.json

FILE_ID=$(python3 - <<'PY'
import json
print(json.load(open('/tmp/devjur-upload-response.json'))['fileId'])
PY
)

curl -sS http://localhost:8080/api/platform/files/$FILE_ID \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN"

curl -sS http://localhost:8080/api/platform/files/$FILE_ID/download \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN" \
  -o /tmp/devjur-download.txt

cmp -s /tmp/devjur-upload.txt /tmp/devjur-download.txt && echo "download ok"
```

Valide a auditoria tenant-local:

```bash
docker compose -f devjur-platform-infra/docker-compose.yml exec postgres-tenant-demo \
  psql -U devjur_tenant_demo -d devjur_tenant_demo \
  -c "select action, resource_type, resource_id, actor, occurred_at from audit_log where resource_id = '$FILE_ID' order by occurred_at;"
```

A consulta deve retornar `FILE_UPLOADED` e `FILE_DOWNLOADED`.

### Mailpit

Acesse:

```text
http://localhost:8025/
```

Smoke test SMTP direto:

```bash
python3 - <<'PY'
import smtplib
from email.message import EmailMessage

msg = EmailMessage()
msg['From'] = 'no-reply@devjur.local'
msg['To'] = 'admin.demo@devjur.local'
msg['Subject'] = 'DEVJUR Mailpit smoke'
msg.set_content('foundation-v4 mailpit smoke validation')

with smtplib.SMTP('localhost', 1025, timeout=10) as smtp:
    smtp.send_message(msg)
PY

curl -sS http://localhost:8025/api/v1/messages
```

O backend possui `PlatformMailService`; nesta etapa ainda nao ha endpoint HTTP dedicado para disparar email pela API.

### Observabilidade

Valide health e Prometheus:

```bash
curl -sS http://localhost:8080/actuator/health

curl -sS http://localhost:8080/actuator/prometheus \
  -H "Host: demo.api.localhost" \
  -H "Authorization: Bearer $TOKEN" \
  | rg 'devjur_|http_server_requests'
```

Valide portas OTLP:

```bash
curl -sS -o /dev/null -w "%{http_code}\n" http://localhost:4318/
timeout 2 bash -c '</dev/tcp/127.0.0.1/4317' && echo "otlp grpc ok"
```

`http://localhost:4318/` pode retornar `404` na raiz; isso confirma que a porta HTTP esta acessivel. Os dados aparecem no LGTM/Grafana em `http://localhost:3000/`, ou `http://localhost:3010/` quando o override local estiver em uso.

### Outbox, Kafka e Redis

Depois de provisionar `acme`, valide o outbox tenant-local:

```bash
docker compose -f devjur-platform-infra/docker-compose.yml exec -T postgres-tenant-demo \
  psql -U devjur_tenant_demo -d devjur_tenant_demo \
  -c "select event_type, status, attempts, published_at is not null as published from outbox_event order by created_at;"
```

Com `devjur.events.publishing.enabled=true`, o evento deve mudar para `PUBLISHED`.

Valide o topico Kafka:

```bash
docker compose -f devjur-platform-infra/docker-compose.yml exec -T kafka \
  /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

Consuma uma mensagem:

```bash
docker compose -f devjur-platform-infra/docker-compose.yml exec -T kafka \
  /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic devjur.platform.tenant.provisioned.v1 \
  --from-beginning \
  --max-messages 1 \
  --timeout-ms 10000
```

Com `devjur.cache.redis.enabled=true`, chamadas ao host `demo.api.localhost` devem gerar chaves Redis:

```bash
docker compose -f devjur-platform-infra/docker-compose.yml exec -T redis redis-cli keys '*'
```

## Testes

### Contracts

```bash
python3 - <<'PY'
import json, yaml
from pathlib import Path
yaml.safe_load(Path('devjur-contracts/openapi/devjur-platform-api.yaml').read_text())
yaml.safe_load(Path('devjur-contracts/asyncapi/devjur-platform-events.yaml').read_text())
for path in Path('devjur-contracts/schemas/events').glob('*.json'):
    json.loads(path.read_text())
print('contracts ok')
PY
```

### Infra

```bash
docker compose -f devjur-platform-infra/docker-compose.yml config >/tmp/devjur-compose.yml
```

### API

```bash
cd devjur-platform-api
mvn test
```

### Web

```bash
cd devjur-platform-web
npm run build
npm test -- --watch=false --browsers=ChromeHeadless
```

## Parar Ambiente

Pare API e Web nos terminais em que estiverem rodando com `Ctrl+C`.

Depois derrube a infraestrutura:

```bash
docker compose -f devjur-platform-infra/docker-compose.yml down
```

Se usou o override local, pare com os mesmos arquivos:

```bash
docker compose \
  -f devjur-platform-infra/docker-compose.yml \
  -f devjur-platform-infra/docker-compose.local-ports.yml \
  down
```

Confirme que as portas foram liberadas:

```bash
ss -ltnp | rg ':1025|:3000|:4200|:4242|:4317|:4318|:5433|:5434|:5435|:5436|:5437|:6379|:8025|:8080|:8081|:8082|:8083|:8200|:9000|:9001|:9092' || true
```

## Branches

A branch atual de trabalho nos quatro repositorios e:

```text
foundation-v4
```

Ela e usada nos quatro repositorios:

- `devjur-contracts`
- `devjur-platform-infra`
- `devjur-platform-api`
- `devjur-platform-web`
