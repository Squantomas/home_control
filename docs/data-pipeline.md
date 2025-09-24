# Data Pipeline

Defines how telemetry flows from devices into InfluxDB for the home control system.

## Sources
- **Shelly Pro 3EM 120A**: Real-time power metrics (voltage, current, power, energy)
- **Shelly Pro 3**: Relay state changes and power consumption
- Future: Additional IoT devices and sensors

## Data Ingestion Methods

### Primary: MQTT (Recommended)
- **Real-time streaming** via MQTT broker (Mosquitto)
- **Low latency**: Sub-second data delivery
- **Event-driven**: Devices publish on state/measurement changes
- **Efficient**: Reduced network overhead vs polling
- **Topics**:
  - `shellies/shellyproplus3-{device_id}/rpc` - Pro 3 relay events
  - `shellies/shellyplusem3-{device_id}/events/rpc` - Pro 3EM measurements

### Fallback: HTTP Polling
- **Backup method** when MQTT unavailable
- **Configurable intervals**: 5-30 seconds
- **REST API calls** to device endpoints
- **Automatic failover** from MQTT when connection lost

### Hybrid Mode
- **MQTT for real-time data** + **HTTP for periodic validation**
- **Data reconciliation** to detect missed messages
- **Enhanced reliability** for critical measurements

## Data Processing Pipeline

### 1. Message Reception
- **MQTT Subscriber**: Listen to device topics with QoS 1 for delivery guarantee
- **HTTP Poller**: Fallback service with configurable intervals
- **Connection Monitoring**: Auto-reconnect and health checks

### 2. Data Transformation
- **Parse JSON payloads** from Shelly MQTT messages
- **Normalize units**: W, V, A, kWh across different message formats
- **Add metadata**: timestamp, device_id, phase, location tags
- **Calculate derivatives**: power trends, energy deltas, efficiency metrics
- **Data validation**: range checks, outlier detection

### 3. Buffering & Batching
- **In-memory buffer** for high-frequency MQTT messages
- **Batch writes** to InfluxDB (configurable batch size/time)
- **Overflow handling**: disk spooling for network outages

## Data Storage (InfluxDB)
- **Bucket**: `home_energy`
- **Retention**: 30–90 days (configurable)
- **Measurements**:
  - `power` - instantaneous power readings (W)
  - `energy` - cumulative energy consumption (kWh)
  - `voltage` - voltage measurements (V)
  - `current` - current measurements (A)
  - `relay_state` - switch on/off events
- **Tags**: device_id, device_type, phase, location, room

## Reliability & Error Handling

### MQTT Resilience
- **Auto-reconnection** with exponential backoff
- **QoS 1** for guaranteed message delivery
- **Persistent sessions** to handle temporary disconnections
- **Dead letter queue** for failed message processing

### HTTP Fallback
- **Health monitoring** of MQTT connection
- **Automatic switching** to HTTP when MQTT fails
- **Backfill mechanism** to catch missed data during outages

### Data Quality
- **Duplicate detection** across MQTT and HTTP sources
- **Timestamp validation** and clock skew handling  
- **Range validation** for sensor readings
- **Idempotent writes** to prevent data corruption

## Monitoring & Observability
- **Connection metrics**: MQTT broker connectivity, message rates
- **Data pipeline metrics**: ingestion lag, processing rates, error counts
- **Device health**: last seen timestamps, message frequency
- **InfluxDB metrics**: write success rates, query performance
- **Alerting**: MQTT disconnections, data gaps, device offline
