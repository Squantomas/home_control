# Backend API

The backend mediates device control and data access.

## Control Endpoints (proposed)
- POST /api/switch/{channel}/on — body: { source }
- POST /api/switch/{channel}/off — body: { source }
- GET /api/switch/{channel}/status
- GET /api/health — backend health

## Metrics Endpoints
- GET /api/metrics/latest — latest totals
- GET /api/metrics/series?phase=A&range=24h — time series passthrough (optional)

## WebSocket (optional)
- /ws — push relay state updates and device health

## Security
- Session or token-based auth for write operations
- Role: viewer (read), operator (write)
