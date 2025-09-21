# Configuration

All secrets and environment-specific values are configured via environment variables or a secret manager. Do not commit secrets.

## Environment Variables (proposed)
- SHELLY_PRO3_IP
- SHELLY_PRO3_USER
- SHELLY_PRO3_PASSWORD
- SHELLY_PRO3EM_IP
- SHELLY_PRO3EM_USER
- SHELLY_PRO3EM_PASSWORD
- INFLUX_URL
- INFLUX_TOKEN
- INFLUX_ORG
- INFLUX_BUCKET
- GRAFANA_URL
- GRAFANA_EMBED_TOKEN (optional)

## Files
- A .env.example file will be added later without real secrets.

## Secret Handling
- Retrieve secrets into env variables at runtime; never echo them
- Rotate periodically
