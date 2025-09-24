# System Architecture

The home control system is a distributed architecture that enables real-time monitoring and control of Shelly Pro devices through a web interface, with all telemetry stored in InfluxDB and visualized via Grafana dashboards.

## System Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web Browser   │    │  Mobile Device  │    │  Other Clients  │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │ HTTPS/WebSocket
                   ┌─────────────▼──────────────┐
                   │        Web UI              │
                   │  (React/Vue/Angular)       │
                   └─────────────┬──────────────┘
                                 │ REST API + WebSocket
                   ┌─────────────▼──────────────┐
                   │      Backend API           │
                   │   (Node.js/Go/Python)      │
                   │  • Device Control          │
                   │  • Data Processing         │
                   │  • Real-time Events        │
                   └─────┬───────────────┬──────┘
                         │               │
          MQTT Subscribe │               │ HTTP Fallback
                         │               │
       ┌─────────────────▼─┐           ┌─▼─────────────────────┐
       │  MQTT Broker      │           │   Shelly Devices      │
       │  (Mosquitto)      │           │                       │
       │  • Message Queue  │◄──MQTT────┤ • Shelly Pro 3        │
       │  • Authentication │           │ • Shelly Pro 3EM 120A │
       │  • QoS Handling   │           │                       │
       └───────────────────┘           └───────────────────────┘
                         │
              Processed   │
              Data       ▼
            ┌─────────────────────┐
            │     InfluxDB        │
            │  • Time Series DB   │
            │  • Data Retention   │
            │  • Query Engine     │
            └─────────┬───────────┘
                      │ InfluxQL/Flux
            ┌─────────▼───────────┐
            │      Grafana        │
            │  • Dashboards       │
            │  • Alerting         │
            │  • Data Viz         │
            └─────────────────────┘
```

## Core Components

### 1. Web UI (Frontend)
- **Technology**: Single-page application (React/Vue.js)
- **Features**: Real-time relay control, energy monitoring, historical charts
- **Communication**: REST API + WebSocket for live updates
- **Embedded Grafana**: Iframe integration for advanced analytics

### 2. Backend API (Application Server)
- **Technology**: Node.js/Go/Python with REST + WebSocket endpoints
- **Responsibilities**:
  - Device command routing (relay control)
  - MQTT message processing and data transformation
  - Real-time WebSocket events to frontend
  - InfluxDB data ingestion and querying
  - Authentication and authorization
- **Scalability**: Horizontal scaling with load balancer support

### 3. MQTT Broker (Message Queue)
- **Implementation**: Mosquitto MQTT broker
- **Purpose**: Real-time device communication hub
- **Features**:
  - QoS 1 message delivery guarantees
  - Authentication and authorization
  - Persistent sessions for reliability
  - Topic-based routing and filtering
- **Topics**: Device-specific channels for commands and telemetry

### 4. Device Layer (IoT Devices)
- **Shelly Pro 3 (Relay Controller)**:
  - 3-channel relay switching
  - Power consumption monitoring per channel
  - MQTT + HTTP dual connectivity
- **Shelly Pro 3EM 120A (Energy Meter)**:
  - 3-phase power monitoring
  - Real-time voltage, current, power measurements
  - High-frequency data streaming via MQTT

### 5. Data Storage (InfluxDB)
- **Type**: Time-series database optimized for IoT data
- **Schema**: Measurements, tags, and fields for device telemetry
- **Retention**: Configurable data retention policies (30-90 days)
- **Performance**: High-throughput writes, optimized time-based queries

### 6. Visualization (Grafana)
- **Purpose**: Advanced analytics and monitoring dashboards
- **Integration**: Direct InfluxDB connection + embedded panels
- **Features**: Real-time charts, alerting, historical analysis
- **Access**: Embedded in web UI + standalone dashboard access

## Enhanced Data Flow

### Primary Data Path (MQTT)
```
Shelly Device → MQTT Broker → Backend Subscriber → Data Processing → InfluxDB
     ↓              ↓              ↓                    ↓             ↓
• Sensor data  • QoS delivery  • JSON parsing    • Transformation  • Time-series
• State changes • Topic routing • Validation      • Aggregation     • Retention
• Real-time     • Authentication• Error handling  • Enrichment      • Indexing
```

### Fallback Data Path (HTTP)
```
Backend Poller → HTTP Request → Shelly Device → JSON Response → Data Processing → InfluxDB
     ↓              ↓              ↓              ↓                    ↓             ↓
• Scheduled    • REST API      • Device query  • Structured data • Transformation  • Storage
• Health check  • Timeout      • Status check  • Error handling  • Validation      • Backup
```

### Control Flow (User Commands)
```
Web UI → Backend API → MQTT Publish → Shelly Device → State Change → MQTT Event → Real-time Update
  ↓          ↓             ↓              ↓             ↓              ↓              ↓
• User     • Command     • Relay control • Physical    • Confirmation • Event       • UI refresh
• Action   • Validation  • Topic         • Switch      • Message      • Processing  • Feedback
```

### Query Flow (Data Visualization)
```
Grafana Dashboard → InfluxDB Query → Time-series Data → Chart Rendering → Web Display
       ↓                   ↓                ↓                ↓               ↓
• User request      • Flux/InfluxQL   • Historical data • Visualization  • Real-time
• Time range        • Aggregation     • Real-time feed • Chart types    • Auto-refresh
```

## Security & Networking

### Network Architecture
- **Device Network**: Local network segment (192.168.x.x) with static IP assignments
- **MQTT Security**: Username/password authentication with optional TLS encryption
- **API Security**: HTTPS/TLS for all web traffic, JWT tokens for authentication
- **Firewall Rules**: Restrict external access, allow only necessary ports (80, 443, 1883)
- **Device Access**: Local network only, no direct internet exposure

### Credential Management
- **Environment Variables**: All secrets stored in `.env` file (never committed)
- **MQTT Authentication**: Dedicated service account with minimal privileges
- **Database Credentials**: Separate InfluxDB service user for application access
- **Rotation Policy**: Regular credential rotation for production deployments

### Data Privacy
- **Local Processing**: All data processing occurs on local infrastructure
- **No Cloud Dependencies**: System operates completely offline-capable
- **Data Encryption**: TLS in transit, optional encryption at rest for sensitive data

## Reliability & Fault Tolerance

### Connection Resilience
- **MQTT Auto-reconnect**: Exponential backoff with persistent sessions
- **HTTP Fallback**: Automatic switching when MQTT broker unavailable
- **Device Health Monitoring**: Regular ping checks and last-seen tracking
- **Circuit Breaker Pattern**: Temporary failure isolation and recovery

### Data Integrity
- **Idempotent Operations**: Safe to retry database writes and device commands
- **Duplicate Detection**: Message deduplication across MQTT and HTTP sources
- **Data Validation**: Input validation and range checking for all sensor data
- **Backup Strategy**: InfluxDB snapshots and configuration backups

### System Monitoring
- **Health Endpoints**: `/health` API for system status checks
- **Metrics Collection**: Connection states, message rates, error counts
- **Alerting**: Configurable alerts for device offline, data gaps, system errors
- **Logging**: Structured logging with configurable levels (debug, info, warn, error)

### Scalability Considerations
- **Horizontal Scaling**: Backend API can run multiple instances with load balancer
- **Database Sharding**: InfluxDB clustering for high-volume deployments
- **MQTT Clustering**: Mosquitto cluster setup for high-availability
- **Caching Layer**: Redis cache for frequently accessed device states and configurations
