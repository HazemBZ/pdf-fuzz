# pdf-fuzz Setup and Fixes

## Overview

Cloned and ran the [pdf-fuzz](https://github.com/HazemBZ/pdf-fuzz) project — a PoC for bulk fuzzy text search in PDF files. The stack uses Django backend, React frontend, Elasticsearch, RabbitMQ, Celery, and Flower dashboard, all orchestrated with Docker Compose.

## Issues Found and Fixed

### 1. Dashboard container failed to start

**Symptom:** `celery worker` exited with `Error: No such option: --broker`

**Root cause:** Newer Celery versions removed `--broker` as a CLI option on `celery worker`.

**Fix:** Removed `worker` subcommand and `--broker` flag from the dashboard command. Broker is already set via environment variable.

```yaml
# Before
command: "uv run -- celery -A PDF_Fuzz worker --broker=amqp://guest@rabbitmq// flower --loglevel=info"

# After
command: "uv run -- celery -A PDF_Fuzz flower --loglevel=info"
```

### 2. Frontend API requests failed from remote server

**Symptom:** Browser couldn't reach the API when accessing the app from a remote IP.

**Root cause:** `settings.ts` hardcoded `localhost:7000` as the API target. On a remote machine, the browser tries to hit `localhost` on the client, not the server.

**Fix:** Changed to `window.location.host` so the frontend dynamically uses the host it was loaded from.

```typescript
// Before
const targetGateway = "localhost:7000";
export const targetServer = targetGateway;

// After
export const targetServer = window.location.host;
```

### 3. Frontend Docker build failed (pnpm/Node version mismatch)

**Symptom:** `pnpm install` failed with `Error [ERR_UNKNOWN_BUILTIN_MODULE]: No such built-in module: node:sqlite`

**Root cause:** Corepack auto-downloaded pnpm 11, which requires Node 22+, but the Dockerfile uses Node 20. The lockfile (v9.0) is from pnpm 9.

**Fix:** Pinned corepack to pnpm 9 in the Dockerfile.

```dockerfile
# Before
RUN corepack enable

# After
RUN corepack enable && corepack prepare pnpm@9 --activate
```

## Services and Ports

| Service    | Port(s)        | Description                |
|------------|----------------|----------------------------|
| Frontend   | 7000, 8765     | React SPA + nginx proxy    |
| Backend    | 8000, 8888     | Django REST API            |
| Elasticsearch | 9200        | Fuzzy text search index    |
| RabbitMQ   | 5672, 15672    | Message broker             |
| Redis      | 6379           | Celery result backend      |
| Flower     | 5555           | Celery task dashboard (optional, requires `--profile flower`) |

## Access

- App: `http://<server-ip>:7000`
- RabbitMQ Management: `http://<server-ip>:15672` (guest/guest)

### Flower Dashboard (Optional)

Flower is behind a Docker Compose profile and is not started by default.

```bash
# Start with flower
docker compose --profile flower up -d

# Start just flower (after base services are up)
docker compose --profile flower up -d dashboard
```

- Flower: `http://<server-ip>:5555`
