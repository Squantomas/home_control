# Home Control for Shelly Pro 3 and Shelly Pro 3EM 120A

A home control website to operate a Shelly Pro 3 relay and visualize electricity consumption from a Shelly Pro 3EM 120A. All telemetry is collected in InfluxDB and explored via Grafana dashboards embedded in this website.

## Features
- Control Shelly Pro 3 relay channels (on/off, status)
- Ingest Shelly Pro 3EM power metrics into InfluxDB
- Grafana dashboards for energy monitoring
- Web UI to operate relays and view charts

## Architecture
See docs/architecture.md for a high-level overview of components:
- Web UI and backend API
- Device integrations (Shelly Pro 3, Shelly Pro 3EM)
- Data pipeline to InfluxDB
- Grafana visualization

## Documentation Index
Start with docs/README.md for the documentation table of contents.

## Quickstart (high level)
1) Configure device IPs/credentials: see docs/configuration.md
2) Provision InfluxDB on your Ubuntu server (no Docker): see docs/influxdb.md and docs/deployment.md
3) Install Grafana on the same Ubuntu server and connect to InfluxDB: see docs/grafana.md and docs/deployment.md
4) Run the website/backend (when added) as native services via systemd: see docs/deployment.md and docs/website.md

## Repository Structure
- README.md — project overview
- CHANGELOG.md — notable changes
- docs/ — detailed documentation

## Status
Initial scaffolding. See docs/roadmap.md for planned work.
