# Shelly Pro 3 (Relay)

The Shelly Pro 3 provides three relay channels that can be controlled programmatically.

## Prerequisites
- Device reachable on your LAN with a static/reserved IP
- Admin credentials with API access

## Control API (HTTP RPC)
Shelly Pro/Plus devices support JSON-RPC over HTTP. Typical endpoints use /rpc with function names.

Examples (adjust IP, credentials, and channel IDs):
- Turn ON channel 0:
  http://<SHELLY_PRO_3_IP>/rpc/Switch.Set?id=0&on=true
- Turn OFF channel 0:
  http://<SHELLY_PRO_3_IP>/rpc/Switch.Set?id=0&on=false
- Get status for channel 0:
  http://<SHELLY_PRO_3_IP>/rpc/Switch.GetStatus?id=0

If authentication is required, pass credentials as per device configuration (e.g., basic auth or device token). See docs/configuration.md.

## Channel Mapping
- id=0 → Relay 1
- id=1 → Relay 2
- id=2 → Relay 3

## Backend Integration
- Backend exposes REST endpoints to toggle channels safely (see docs/api.md)
- Implement debouncing and optional interlock if channels must be mutually exclusive

## Safety
- Validate load ratings, add failsafes
- Log every state change with actor and timestamp
