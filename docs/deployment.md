# Deployment

Target: Ubuntu production server (no Docker). All components run as native packages/services managed by systemd.

## Prerequisites
- Ubuntu 22.04 LTS or newer with a sudo-capable user
- Public DNS name (optional but recommended) and inbound HTTPS/SSH allowed
- Static IP or reserved DHCP lease for the server

## MQTT Broker (Mosquitto)
Install and configure Mosquitto MQTT broker for real-time device communication:

### Installation
```bash
# Update package list
sudo apt update

# Install Mosquitto broker and client tools
sudo apt install -y mosquitto mosquitto-clients

# Enable and start the service
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

### Configuration
```bash
# Create configuration directory
sudo mkdir -p /etc/mosquitto/conf.d

# Create main configuration file
sudo tee /etc/mosquitto/mosquitto.conf > /dev/null <<EOF
# Basic Configuration
pid_file /var/run/mosquitto.pid
persistence true
persistence_location /var/lib/mosquitto/
log_dest file /var/log/mosquitto/mosquitto.log
include_dir /etc/mosquitto/conf.d

# Network
listener 1883 localhost
protocol mqtt

# Authentication
allow_anonymous false
password_file /etc/mosquitto/passwd

# Security (uncomment for remote access)
# listener 8883
# protocol mqtt
# cafile /etc/mosquitto/ca_certificates/ca.crt
# certfile /etc/mosquitto/certs/server.crt
# keyfile /etc/mosquitto/certs/server.key
EOF

# Create MQTT user for home control system
sudo mosquitto_passwd -c /etc/mosquitto/passwd homecontrol

# Set proper permissions
sudo chown mosquitto:mosquitto /etc/mosquitto/passwd
sudo chmod 600 /etc/mosquitto/passwd

# Create log directory and set permissions
sudo mkdir -p /var/log/mosquitto
sudo chown mosquitto:mosquitto /var/log/mosquitto

# Restart Mosquitto with new configuration
sudo systemctl restart mosquitto

# Verify service status
sudo systemctl status mosquitto
```

### Firewall Configuration
```bash
# Allow MQTT port (if needed for remote device access)
sudo ufw allow 1883/tcp comment 'MQTT'

# For TLS/SSL MQTT (optional)
sudo ufw allow 8883/tcp comment 'MQTT TLS'
```

See [docs/mqtt.md](mqtt.md) for detailed configuration and device setup.

## InfluxDB 2.x (OSS on Ubuntu)
Install and enable InfluxDB 2.x as a native service:

### Installation
```bash
# Import InfluxDB GPG key
curl -s https://repos.influxdata.com/influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg

# Add repository
echo "deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main" | sudo tee /etc/apt/sources.list.d/influxdata.list

# Update package list and install
sudo apt update
sudo apt install -y influxdb2

# Enable and start service
sudo systemctl enable influxdb
sudo systemctl start influxdb
```

### Initial Setup
```bash
# Configure InfluxDB via web interface
# Visit http://<server>:8086
# Create organization: "home_control"
# Create initial bucket: "home_energy" (retention: 90 days)
# Generate API token with read/write permissions
# Store credentials in .env file
```

See [docs/influxdb.md](influxdb.md) for detailed commands and configuration.

## Grafana OSS (Ubuntu)
Install Grafana as a native service and configure it to use InfluxDB as the datasource.

- Install via apt and enable grafana-server
- Access Grafana at http://<server>:3000
- Create a read-only datasource/token for embedding dashboards in the website

See docs/grafana.md for detailed commands and configuration.

## Reverse Proxy (Nginx)
Recommended for TLS termination, access control, and WebSocket support.

### Installation
```bash
# Install Nginx
sudo apt install -y nginx

# Obtain SSL certificates (Let's Encrypt)
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

### Configuration
Create `/etc/nginx/sites-available/home-control`:
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name your-domain.com;

    # SSL Configuration (managed by certbot)
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # Grafana (embedded dashboards)
    location /grafana/ {
        proxy_pass http://127.0.0.1:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support for live dashboards
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # Backend API
    location /api/ {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket endpoint for real-time updates
    location /ws {
        proxy_pass http://127.0.0.1:8080/ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # MQTT over WebSocket (optional for web clients)
    location /mqtt {
        proxy_pass http://127.0.0.1:9001/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

    # Static website files
    location / {
        root /srv/home_control/public;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # Security: deny access to sensitive files
    location ~ /\.(env|git) {
        deny all;
        return 404;
    }
}
```

### Enable Site
```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/home-control /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

## Backend and Website (when added)
The backend service handles MQTT communication, HTTP fallback, and API endpoints.

### Service Installation
```bash
# Create service user
sudo useradd --system --shell /bin/false --home /srv/home_control homecontrol

# Install backend under service directory
sudo mkdir -p /srv/home_control
sudo chown homecontrol:homecontrol /srv/home_control

# Copy application files (via git or rsync)
# Set environment variables in /srv/home_control/.env
sudo -u homecontrol cp .env.example /srv/home_control/.env
sudo -u homecontrol vi /srv/home_control/.env
```

### Systemd Service Configuration
Create `/etc/systemd/system/home-control-backend.service`:
```ini
[Unit]
Description=Home Control Backend API
After=network.target mosquitto.service influxdb.service
Requires=mosquitto.service
Wants=influxdb.service

[Service]
Type=simple
User=homecontrol
Group=homecontrol
WorkingDirectory=/srv/home_control
EnvironmentFile=/srv/home_control/.env
ExecStart=/srv/home_control/bin/backend
Restart=on-failure
RestartSec=5s
TimeoutStopSec=10s

# Security
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/srv/home_control

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=home-control-backend

[Install]
WantedBy=multi-user.target
```

### Service Management
```bash
# Enable and start the backend service
sudo systemctl daemon-reload
sudo systemctl enable home-control-backend
sudo systemctl start home-control-backend

# Check service status
sudo systemctl status home-control-backend

# View logs
sudo journalctl -u home-control-backend -f
```

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
