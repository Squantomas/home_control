# InfluxDB

## Version
Use InfluxDB 2.x (OSS) installed natively on Ubuntu.

## Install on Ubuntu (no Docker)
1) Add repository and key
- Create keyring and add InfluxData repo for Ubuntu
2) Install and enable service
- apt update && apt install influxdb2
- systemctl enable --now influxdb
3) Initial setup
- Open http://<server>:8086 and create org, bucket (e.g., home_energy), and a write token
- Store token securely and configure backend env vars (see docs/configuration.md)

## Buckets
- home_energy (retention 30–90d)

## Line Protocol Fields (examples)
- power, voltage, current per phase
- total_power, energy_kwh

Example line protocol (illustrative):
energy,device_id=pro3em,phase=A power=123.4,voltage=229.7,current=0.54 1690000000000

## Authentication
- Use a token via environment variables; never commit secrets
- Configure org and bucket via env

## Write Path
- HTTP API /api/v2/write with precision=s

## Querying
- Use Flux in Grafana or InfluxDB UI
- Example Flux (per-phase power, last 24h):
from(bucket: "home_energy")
  |> range(start: -24h)
  |> filter(fn: (r) => r._measurement == "energy" and r.phase == "A" and r._field == "power")
  |> aggregateWindow(every: 1m, fn: mean, createEmpty: false)
  |> yield(name: "mean")

## Maintenance
- Backups of /var/lib/influxdb2 and config
- Token rotation
- Retention tuning
