# Shelly Pro 3EM 120A (Energy Meter)

Three-phase energy meter providing per-phase voltage, current, power, and total energy.

## Prerequisites
- Device reachable on your LAN (static IP recommended)
- Admin credentials with API access

## Telemetry API (HTTP RPC)
Shelly Pro 3EM supports telemetry via JSON-RPC. A common status function:
- Get status (all phases):
  http://<SHELLY_PRO_3EM_IP>/rpc/EM.GetStatus

Example response includes per-phase measurements (voltage, current, power), total active power, cumulative energy, etc. Refer to the official device docs for the full schema.

## Ingestion Strategy
Two options:
1) Polling: Backend polls /rpc/EM.GetStatus at a fixed interval (e.g., 5-10s)
2) MQTT: Subscribe to device topics if MQTT is enabled (lower latency)

Start with polling for simplicity; consider MQTT later. See docs/data-pipeline.md.

## Measurements → InfluxDB
- Measurement: power, voltage, current, energy
- Tags: device_id, phase (A|B|C), site, room
- Fields: numeric values (float), totals

## Validation
- Handle missing values and transient errors
- Normalize units (W, V, A, kWh)
