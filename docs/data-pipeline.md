# Data Pipeline

Defines how telemetry flows from devices into InfluxDB.

## Sources
- Shelly Pro 3EM 120A: power metrics via HTTP RPC or MQTT
- (Optional) Shelly Pro 3 relay state telemetry for audit

## Transport
- v1: HTTP polling every 5–10 seconds
- v2: MQTT subscription (optional enhancement)

## Transform
- Normalize fields (W, V, A, kWh)
- Attach tags: device_id, phase, site, room
- Derive totals (sum of phases) if needed

## Sink (InfluxDB)
- Bucket: home_energy
- Retention: 30–90 days (tunable)
- Schema-on-write: measurement names and tags as above

## Backpressure & Reliability
- Bounded queue with retries and exponential backoff
- Drop or flag outliers
- Idempotent writes

## Observability
- Metrics for ingestion lag, success rates
- Logs for device/API errors
