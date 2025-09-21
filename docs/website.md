# Website / UI

## Goals
- Control Shelly Pro 3 relay channels
- Display Grafana dashboards for energy metrics

## Initial Pages
- / — Overview and quick controls
- /controls — Relay controls (ch0–ch2)
- /energy — Embedded Grafana panels

## Embedding Grafana
- Iframe URL with orgId, panelId, and kiosk mode
- Protect with reverse proxy and viewer token/session

## UX Guidelines
- Show safe state transitions (pending → success/fail)
- Disable controls if backend reports device offline
- Provide event log for state changes
