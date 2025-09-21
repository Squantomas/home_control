# Architecture

This system connects a home control web app to Shelly Pro devices, collects metrics in InfluxDB, and visualizes them with Grafana.

## Components
- Web UI: Single-page UI to operate relays and view charts
- Backend API: REST/WS service for device control and data queries
- Device Integrations:
  - Shelly Pro 3 (relay): HTTP RPC for switching channels
  - Shelly Pro 3EM 120A (meter): HTTP RPC or MQTT for telemetry
- Data Store: InfluxDB (time-series)
- Visualization: Grafana dashboards (embedded)

## Data Flow (reference)
1) Backend polls or subscribes to Shelly Pro 3EM metrics
2) Backend writes data points to InfluxDB
3) Grafana queries InfluxDB to render dashboards
4) UI embeds Grafana panels and calls backend to switch relays

## Security & Networking
- Local network access to devices (static IPs recommended)
- Backend credentials stored via environment variables; never commit secrets
- Use HTTPS/TLS for UI/API when exposed beyond LAN

## Reliability
- Retries/backoff for device calls
- Idempotent writes to InfluxDB
- Health endpoints and logs for monitoring
