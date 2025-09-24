# MQTT Broker Setup and Configuration

This document covers setting up and configuring MQTT for real-time data streaming from Shelly devices.

## Overview

MQTT (Message Queuing Telemetry Transport) provides real-time, efficient communication between your Shelly devices and the home control system. This replaces HTTP polling with event-driven messaging.

### Benefits
- **Real-time data**: Sub-second latency for device events
- **Efficient**: Reduced network traffic vs polling
- **Reliable**: QoS guarantees and persistent sessions
- **Scalable**: Easy to add new devices and subscribers

## MQTT Broker Installation

### Mosquitto MQTT Broker (Ubuntu)

Install Mosquitto on your Ubuntu server:

```bash
# Update package list
sudo apt update

# Install Mosquitto broker and clients
sudo apt install -y mosquitto mosquitto-clients

# Enable and start the service
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

### Configuration Files

#### `/etc/mosquitto/mosquitto.conf`

```conf
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

# Security (optional - for remote access)
# listener 8883
# protocol mqtt
# cafile /etc/mosquitto/ca_certificates/ca.crt
# certfile /etc/mosquitto/certs/server.crt
# keyfile /etc/mosquitto/certs/server.key
```

#### Create User Authentication

```bash
# Create password file
sudo touch /etc/mosquitto/passwd

# Add user (replace 'homecontrol' with your preferred username)
sudo mosquitto_passwd /etc/mosquitto/passwd homecontrol

# Set proper permissions
sudo chown mosquitto:mosquitto /etc/mosquitto/passwd
sudo chmod 600 /etc/mosquitto/passwd

# Restart Mosquitto
sudo systemctl restart mosquitto
```

## Shelly Device Configuration

### Shelly Pro 3 (Relay)

Configure MQTT in the Shelly Pro 3 web interface:

1. **Access device**: `http://<SHELLY_PRO3_IP>`
2. **Navigate**: Settings → Networks → MQTT
3. **Configuration**:
   - **Enable**: ✓ Enable MQTT
   - **Server**: `<MQTT_BROKER_HOST>:1883`
   - **Username**: `homecontrol`
   - **Password**: `<your_mqtt_password>`
   - **Client ID**: `shellyproplus3-<device_id>`
   - **Keep alive**: 60 seconds
   - **Clean session**: ✓ Enabled
   - **Retain**: ✓ Enabled for status topics

### Shelly Pro 3EM 120A (Energy Meter)

Configure MQTT in the Shelly Pro 3EM web interface:

1. **Access device**: `http://<SHELLY_PRO3EM_IP>`
2. **Navigate**: Settings → Networks → MQTT
3. **Configuration**:
   - **Enable**: ✓ Enable MQTT
   - **Server**: `<MQTT_BROKER_HOST>:1883`
   - **Username**: `homecontrol`
   - **Password**: `<your_mqtt_password>`
   - **Client ID**: `shellyplusem3-<device_id>`
   - **Keep alive**: 60 seconds
   - **Report interval**: 10 seconds (or desired frequency)

## MQTT Topics Structure

### Shelly Pro 3 (Relay) Topics

```
shellies/shellyproplus3-<device_id>/
├── online                    # Device online status (true/false)
├── events/rpc               # RPC events and responses  
├── status/switch:0          # Relay 0 status
├── status/switch:1          # Relay 1 status
├── status/switch:2          # Relay 2 status
├── command/switch:0/set     # Control relay 0 (publish here)
├── command/switch:1/set     # Control relay 1 (publish here)
└── command/switch:2/set     # Control relay 2 (publish here)
```

### Shelly Pro 3EM 120A Topics

```
shellies/shellyplusem3-<device_id>/
├── online                   # Device online status
├── events/rpc              # Measurement events
├── status/em3:0            # Energy meter status (all phases)
├── status/emdata:0         # Real-time energy data
└── status/temperature      # Device temperature
```

## Message Formats

### Relay State (Pro 3)
```json
{
  "id": 0,
  "output": true,
  "source": "WS_in",
  "timestamp": 1693824000
}
```

### Energy Measurement (Pro 3EM)
```json
{
  "id": 0,
  "a_current": 1.23,
  "a_voltage": 230.5,
  "a_act_power": 283.4,
  "a_aprt_power": 15.2,
  "b_current": 0.85,
  "b_voltage": 229.8,
  "b_act_power": 195.3,
  "b_aprt_power": 12.1,
  "c_current": 2.10,
  "c_voltage": 231.2,
  "c_act_power": 485.5,
  "c_aprt_power": 28.3,
  "total_current": 4.18,
  "total_act_power": 964.2,
  "total_aprt_power": 55.6,
  "timestamp": 1693824015
}
```

## Testing MQTT Connection

### Verify Broker Status

```bash
# Check Mosquitto service
sudo systemctl status mosquitto

# Check logs
sudo tail -f /var/log/mosquitto/mosquitto.log

# Test local connection
mosquitto_pub -h localhost -p 1883 -u homecontrol -P <password> -t test/topic -m "Hello World"
mosquitto_sub -h localhost -p 1883 -u homecontrol -P <password> -t test/topic
```

### Monitor Device Messages

```bash
# Subscribe to all Shelly messages
mosquitto_sub -h localhost -p 1883 -u homecontrol -P <password> -t "shellies/+/+/+"

# Subscribe to specific device
mosquitto_sub -h localhost -p 1883 -u homecontrol -P <password> -t "shellies/shellyproplus3-<device_id>/+"

# Monitor energy measurements
mosquitto_sub -h localhost -p 1883 -u homecontrol -P <password> -t "shellies/shellyplusem3-<device_id>/status/emdata:0"
```

### Send Relay Commands

```bash
# Turn on relay 0
mosquitto_pub -h localhost -p 1883 -u homecontrol -P <password> \
  -t "shellies/shellyproplus3-<device_id>/command/switch:0/set" \
  -m '{"on": true}'

# Turn off relay 1  
mosquitto_pub -h localhost -p 1883 -u homecontrol -P <password> \
  -t "shellies/shellyproplus3-<device_id>/command/switch:1/set" \
  -m '{"on": false}'
```

## Troubleshooting

### Common Issues

#### Connection Refused
- Check Mosquitto service: `sudo systemctl status mosquitto`
- Verify credentials: `mosquitto_pub -h localhost -u homecontrol -P <password> -t test -m test`
- Check firewall: `sudo ufw status`

#### No Messages from Devices
- Verify device MQTT configuration
- Check device connectivity: `ping <device_ip>`
- Monitor broker logs: `sudo tail -f /var/log/mosquitto/mosquitto.log`
- Check device logs via web interface

#### Authentication Failures
- Verify password file: `sudo cat /etc/mosquitto/passwd`
- Test credentials: `mosquitto_sub -h localhost -u homecontrol -P <password> -t '$SYS/broker/version'`
- Check permissions: `ls -la /etc/mosquitto/passwd`

### Performance Tuning

#### High Message Volume
```conf
# Add to /etc/mosquitto/mosquitto.conf
max_connections 100
max_inflight_messages 100
max_queued_messages 1000
message_size_limit 1024
```

#### Memory Optimization
```conf
# Persistence settings
autosave_interval 60
autosave_on_changes false
persistent_client_expiration 1h
```

## Security Considerations

### Network Security
- Use TLS/SSL for remote access (port 8883)
- Restrict broker access to local network
- Configure firewall rules appropriately

### Authentication
- Use strong passwords for MQTT users
- Rotate credentials regularly
- Consider certificate-based authentication for production

### Device Security
- Keep Shelly firmware updated
- Use strong WiFi passwords
- Consider device network isolation (VLAN)

## Integration with Home Control Backend

The home control backend will:

1. **Connect** to MQTT broker using credentials from `.env`
2. **Subscribe** to device topics for real-time data
3. **Parse** JSON messages and transform to InfluxDB format
4. **Publish** relay control commands when requested via API
5. **Monitor** connection health and fallback to HTTP if needed

See [Data Pipeline](data-pipeline.md) for detailed processing flow.