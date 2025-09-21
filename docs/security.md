# Security

## Principles
- Least privilege for tokens and roles
- No secrets in source control
- HTTPS/TLS when exposed beyond LAN

## Device Security
- Change default passwords
- Restrict device access to trusted VLAN/subnet

## Backend Security
- Validate all inputs to control endpoints
- Rate limit switching operations
- Audit log for state changes

## Grafana/InfluxDB
- Viewer-only credentials for embedding
- Separate write token for ingestion
