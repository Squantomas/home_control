# Operations Runbook

## Monitoring
- Backend health endpoint
- Ingestion lag metrics

## Service Management (Ubuntu)
- Check status: systemctl status influxdb grafana-server <backend_service>
- Restart services: systemctl restart influxdb grafana-server <backend_service>
- View logs: journalctl -u influxdb -u grafana-server -u <backend_service> -n 200 -f

## Incident Response
- Device unreachable: check network, power, credentials
- InfluxDB down: restart service, verify disk space
- Grafana panels blank: verify datasource/token

## Maintenance
- Token rotation schedule
- Firmware updates for devices (planned windows)
