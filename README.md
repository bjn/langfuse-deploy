# langfuse-deploy

Shared Langfuse v3 observability stack for the home-lab Coolify environment.

## Purpose

This stack runs Langfuse as a dedicated service instead of embedding it into Jeeves. Jeeves, Hermes, Hindsight, and agent services can all point at the same Langfuse instance.

## Public UI

- URL: `https://langfuse.crankingoutcode.com`
- Protected by the existing Authentik Traefik forward-auth outpost.

## Internal service URLs

Services attached to the app networks can use either:

- `http://langfuse:3000`
- `http://langfuse-web:3000`

Current attached external networks:

- Hermes agent network: `treg5ilrtlvhiuw90tql5y4l`
- Jeeves network: `g4cw8o8skg04o0kwo4wk0cso`
- Hindsight memory network: `xtlb8ebit5pky5e5wlpotbbw`

If a Coolify app is recreated and gets a new Docker network name, update `docker-compose.yml` and redeploy this stack.

## Required Coolify env vars

Set these in the Coolify resource; do not commit real values:

```env
NEXTAUTH_URL=https://langfuse.crankingoutcode.com
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<secret>
POSTGRES_DB=postgres
CLICKHOUSE_USER=clickhouse
CLICKHOUSE_PASSWORD=<secret>
MINIO_ROOT_USER=<secret>
MINIO_ROOT_PASSWORD=<secret>
REDIS_AUTH=<secret>
SALT=<secret>
ENCRYPTION_KEY=<64 hex chars from openssl rand -hex 32>
NEXTAUTH_SECRET=<secret>
TELEMETRY_ENABLED=false
```

Optional one-shot bootstrap variables for first deploy:

```env
LANGFUSE_INIT_ORG_ID=home-lab
LANGFUSE_INIT_ORG_NAME=Home Lab
LANGFUSE_INIT_PROJECT_ID=jeeves
LANGFUSE_INIT_PROJECT_NAME=Jeeves
LANGFUSE_INIT_PROJECT_PUBLIC_KEY=<pk-lf-...>
LANGFUSE_INIT_PROJECT_SECRET_KEY=<sk-lf-...>
LANGFUSE_INIT_USER_EMAIL=<email>
LANGFUSE_INIT_USER_NAME=<name>
LANGFUSE_INIT_USER_PASSWORD=<secret>
```

After initial bootstrap, prefer creating additional projects/API keys from the Langfuse UI.

## Consumer app env vars

For apps on one of the internal Docker networks:

```env
LANGFUSE_BASE_URL=http://langfuse:3000
LANGFUSE_HOST=http://langfuse:3000
LANGFUSE_PUBLIC_KEY=<project public key>
LANGFUSE_SECRET_KEY=<project secret key>
```

For clients outside Coolify/Docker, use:

```env
LANGFUSE_BASE_URL=https://langfuse.crankingoutcode.com
LANGFUSE_HOST=https://langfuse.crankingoutcode.com
```

## Notes

- Langfuse components: web, worker, Postgres, ClickHouse, Redis, MinIO.
- Only the web UI is routed publicly through Traefik.
- Data volumes are managed by Docker/Coolify.
- Keep app instrumentation optional so services degrade gracefully if Langfuse is unavailable.
