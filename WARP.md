# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

Project state
- This repository is currently a documentation scaffold for a home control system that will operate a Shelly Pro 3 relay and ingest power metrics from a Shelly Pro 3EM 120A into InfluxDB, with Grafana-based visualization embedded in a website.
- Source code for the backend and web UI is not yet present. See docs/roadmap.md for planned work and docs/deployment.md for environment notes.
- Deployment preference: Ubuntu production server (no Docker); install native services and manage with systemd.

Commands
- Build: not applicable yet (no source code or build tooling is defined in the repo).
- Lint/format: not applicable yet.
- Tests: not applicable yet.
- Action for future updates: once backend/UI code and tooling are added (e.g., Makefile, package.json, docker-compose.yml, or language-specific configs), update this section with concrete build/lint/test commands and how to run a single test.

High-level architecture (from docs)
- Components:
  - Web UI: Single-page interface to operate relays and view charts (docs/website.md)
  - Backend API: REST/WS service mediating device control and data access (docs/api.md)
  - Device integrations:
    - Shelly Pro 3 (relay): control via HTTP RPC
    - Shelly Pro 3EM 120A (meter): telemetry via HTTP polling initially; MQTT optional later (docs/architecture.md, docs/data-pipeline.md)
  - Data store: InfluxDB 2.x for time-series telemetry (docs/influxdb.md)
  - Visualization: Grafana dashboards, embedded into the website (docs/grafana.md)
- Data flow (reference):
  1) Backend polls or subscribes to 3EM telemetry
  2) Backend writes points to InfluxDB (bucket: home_energy; retention 30–90d)
  3) Grafana queries InfluxDB to render dashboards
  4) UI embeds Grafana panels and calls backend to switch relays
- Security & networking:
  - Local network access to devices (static IPs recommended)
  - Backend credentials via environment variables; do not commit secrets
  - HTTPS/TLS when exposed beyond LAN (docs/security.md)
- Reliability considerations:
  - Retries/backoff for device calls, idempotent InfluxDB writes
  - Health endpoints and logs for monitoring (docs/runbook.md)

Configuration (planned)
- Environment variables referenced in docs/configuration.md:
  - SHELLY_PRO3_IP, SHELLY_PRO3_USER, SHELLY_PRO3_PASSWORD
  - SHELLY_PRO3EM_IP, SHELLY_PRO3EM_USER, SHELLY_PRO3EM_PASSWORD
  - INFLUX_URL, INFLUX_TOKEN, INFLUX_ORG, INFLUX_BUCKET
  - GRAFANA_URL, GRAFANA_EMBED_TOKEN (optional)
- A .env.example may be added later without real secrets.

Key docs
- Overview: README.md, docs/README.md
- Architecture: docs/architecture.md
- Data pipeline: docs/data-pipeline.md
- InfluxDB and Grafana specifics: docs/influxdb.md, docs/grafana.md
- Website and API outlines: docs/website.md, docs/api.md
- Operations and controls: docs/runbook.md, docs/controls.md
- Roadmap: docs/roadmap.md

When code is added
- Prefer Ubuntu-native workflows (no Docker):
  - Install dependencies (InfluxDB, Grafana) via apt and manage via systemd
  - Provide systemd unit files for backend and website
  - Lint/format commands (once tooling exists)
  - Test commands, including how to run an individual test
- Deployment: push changes to the Ubuntu server (git-based or rsync) and restart services; see docs/deployment.md
- After those land, update the Commands section above with exact invocations found in the repository tooling.
