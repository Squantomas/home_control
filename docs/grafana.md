# Grafana

## Objective
Dashboards for real-time and historical power consumption.

## Install on Ubuntu (no Docker)
1) Add Grafana OSS repo and keyring
2) Install and enable service
- apt update && apt install grafana
- systemctl enable --now grafana-server
3) Access
- http://<server>:3000 (set admin password on first login)

## Data Source
- InfluxDB (Flux). Configure a read-only datasource that points to the InfluxDB 2.x OSS instance on the same server.

## Panels (initial)
- Total Power Now (stat)
- Per-Phase Power (time series)
- Voltage and Current per Phase (time series)
- Daily Energy (bar/area)

## Embedding in Website
- Iframe panels with auth proxy (recommended) or viewer token
- Protect with reverse proxy and viewer-only credentials

## Access Control
- Read-only viewer for website embedding
- Do not embed admin sessions

## Export/Import
- Store dashboard JSON in repo later (not yet added)
