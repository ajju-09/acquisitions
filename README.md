# Dockerized Neon setup: development (Neon Local) and production (Neon Cloud)

This repo provides a Docker-based setup for running your app with Neon in two modes:

- Development: Neon Local proxy in Docker with ephemeral branches
- Production: App only; connects to Neon Cloud via DATABASE_URL

Prereqs

- Docker and Docker Compose v2+
- Neon account, API key, and Project ID

Files overview

- Dockerfile — multi-stage Node.js image (adjust if your stack differs)
- docker-compose.dev.yml — app + Neon Local (ephemeral branches)
- docker-compose.prod.yml — app only (uses Neon Cloud)
- .env.development — local variables for Neon Local
- .env.production — production variables for Neon Cloud

Environment variables

- NEON_API_KEY: Create in Neon Console → Project Settings → API Keys
- NEON_PROJECT_ID: Project Settings → General
- PARENT_BRANCH_ID: The parent branch to fork ephemeral branches from (e.g., main branch id)
- NEON_DATABASE: DB name exposed by Neon Local (default appdb)
- DATABASE_URL: Standard Postgres connection string

Development (Neon Local + ephemeral branches)

1. Configure .env.development
   NEON_API_KEY=...
   NEON_PROJECT_ID=...
   PARENT_BRANCH_ID=... # parent branch id for ephemeral branches
   NEON_DATABASE=appdb

   # Optionally override the default connection string

   # DATABASE_URL=postgres://neon:npg@neon-local:5432/appdb?sslmode=require

2. Start services
   docker compose -f docker-compose.dev.yml up --build
   - The neon-local service creates an ephemeral branch when it starts and deletes it on stop.
   - The app service sees DATABASE_URL pointing to neon-local:5432.

3. Stopping and cleaning
   docker compose -f docker-compose.dev.yml down
   - This removes the ephemeral branch automatically.

Notes for JS apps (pg / postgres libraries)

- Neon Local uses a self-signed cert. Add SSL config in your code if you connect directly with pg/postgres:
  - pg: pass ssl: { rejectUnauthorized: false }
  - If you cannot modify code, uncomment NODE_TLS_REJECT_UNAUTHORIZED=0 in docker-compose.dev.yml (not recommended for prod).

Production (Neon Cloud)

1. Configure .env.production
   DATABASE_URL=postgres://<user>:<password>@<hostname>.neon.tech/<db>?sslmode=require

2. Start app
   docker compose -f docker-compose.prod.yml up --build -d
   - No Neon Local proxy runs in production.
   - Secrets are injected via env vars; do not hardcode values in images.

Switching between dev and prod

- Dev: docker compose -f docker-compose.dev.yml ... reads .env.development
- Prod: docker compose -f docker-compose.prod.yml ... reads .env.production

Customize for non-Node stacks

- Update Dockerfile base image and start command to match your runtime.
- Ensure your app reads DATABASE_URL from the environment.

Troubleshooting

- Authentication or 403: verify NEON_API_KEY and NEON_PROJECT_ID.
- Cannot connect to DB locally: ensure neon-local is healthy and port 5432 is not in use.
- SSL errors in dev: add rejectUnauthorized: false (dev only) or set NODE_TLS_REJECT_UNAUTHORIZED=0 for local.
