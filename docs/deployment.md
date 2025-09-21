# Deployment

Target: Ubuntu production server (no Docker). All components run as native packages/services managed by systemd.

## Prerequisites
- Ubuntu 22.04 LTS or newer with a sudo-capable user
- Public DNS name (optional but recommended) and inbound HTTPS/SSH allowed
- Static IP or reserved DHCP lease for the server

## InfluxDB 2.x (OSS on Ubuntu)
Install and enable InfluxDB 2.x as a native service:

1) Add repository and install
- Import key and repo
- Install influxdb2
- Enable and start service

2) Initial setup
- Visit http://<server>:8086 to set up org, bucket, and token
- Store token securely; configure env vars for the backend (see docs/configuration.md)

See docs/influxdb.md for detailed commands and configuration.

## Grafana OSS (Ubuntu)
Install Grafana as a native service and configure it to use InfluxDB as the datasource.

- Install via apt and enable grafana-server
- Access Grafana at http://<server>:3000
- Create a read-only datasource/token for embedding dashboards in the website

See docs/grafana.md for detailed commands and configuration.

## Reverse Proxy (Nginx)
Recommended for TLS termination and access control.

- Install Nginx (apt)
- Obtain certificates (Let’s Encrypt or provided certs)
- Configure server blocks to proxy:
  - /grafana → http://127.0.0.1:3000
  - /api and /ws (future backend) → http://127.0.0.1:<backend_port>
  - / (website) → static site or backend UI

## Backend and Website (when added)
- Install backend and website on the Ubuntu server under /srv/home_control
- Run both as systemd services owned by dedicated service users (not root)
- Provide unit files (e.g., /etc/systemd/system/home-control-backend.service) with Restart=on-failure
- Log via journald and rotate using the system defaults

## Deploying Changes to the Ubuntu Server
Choose one approach; both avoid Docker and run natively.

Option A: Git-based deploy (recommended)
- Create a bare repo on the server (e.g., /srv/git/home_control.git) with a post-receive hook that checks out to /srv/home_control
- Push from local: git push production main
- The hook can run systemctl daemon-reload and systemctl restart <services>

Option B: Rsync
- From local, sync the repo to the server using rsync over SSH (excluding .git)
- After sync, reload services: systemctl restart <services>

## Backups and Persistence
- InfluxDB: back up data directory (/var/lib/influxdb2) and config
- Grafana: back up /var/lib/grafana and /etc/grafana
- Test restores periodically

## Notes
- No Docker or containers are used in this deployment plan
- Use firewall rules (ufw) to expose only SSH and HTTPS
