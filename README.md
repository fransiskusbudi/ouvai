# ouvai

Self-hosted observability stack for the [atoue](https://atoue.io) projects. Runs on a single VPS with Docker Compose.

## What's inside

- **Prometheus** — scrapes host and application metrics
- **Grafana** — dashboards for system health and application usage (port 3000, localhost-only)
- **node-exporter** — host-level metrics (CPU, memory, disk, network)
- **nginx** — reverse proxy with TLS for `monitoring.atoue.io`

## Dashboards

- **Node Exporter Full** — VPS health (CPU, memory, disk, network, load)
- **Taliu Application Metrics** — usage analytics for [taliu](https://taliu.atoue.io):
  - Sessions, messages, tokens, and estimated cost (split by text vs voice channel)
  - Voice metrics row: call count, duration, latency, STT/TTS cost breakdown
  - Recent sessions and recent message contents

## Architecture

```
                ┌─────────────┐
                │ Cloudflare  │
                └──────┬──────┘
                       │ HTTPS
                ┌──────▼──────┐
                │    nginx    │   monitoring.atoue.io
                └──────┬──────┘
                       │
                ┌──────▼──────┐    ┌─────────────────┐
                │   Grafana   │◄───┤  taliu-postgres │
                └──────┬──────┘    └─────────────────┘
                       │                   ▲
                ┌──────▼──────┐            │  (joined via docker network)
                │ Prometheus  │
                └──────┬──────┘
                       │
                ┌──────▼──────┐
                │node-exporter│
                └─────────────┘
```

Grafana connects to the taliu Postgres container directly through the shared `taliu_agent-network` Docker network, so Postgres can stay locked to `127.0.0.1` on the host.

## Setup

```bash
cp .env.example .env  # fill in GF_SECURITY_ADMIN_PASSWORD and POSTGRES_PASSWORD
docker compose up -d
```

Grafana provisioning auto-loads dashboards and datasources from `grafana/provisioning/`.

## Configuration

Environment variables (see `.env`):

| Var | Purpose |
|-----|---------|
| `GF_SECURITY_ADMIN_USER` | Grafana admin username |
| `GF_SECURITY_ADMIN_PASSWORD` | Grafana admin password |
| `GF_SERVER_ROOT_URL` | Public URL Grafana renders links with |
| `GF_AUTH_ANONYMOUS_ENABLED` | Set to `false` |
| `POSTGRES_PASSWORD` | Read by the Grafana datasource provisioning |

## Related repos

- [atoue.io](https://github.com/fransiskusbudi/atoue.io) — landing page
- [taliu](https://github.com/fransiskusbudi/taliu) — AI resume agent (the application being monitored)
