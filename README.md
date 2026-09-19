# Cloud-Native IoT Telemetry & Observability Platform (Helm / GitOps)

![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5.svg)
![Helm](https://img.shields.io/badge/Packaging-Helm%20v3-0F1689.svg)
![LoRaWAN](https://img.shields.io/badge/Network-ChirpStack%20v4-4A90E2.svg)
![TSDB](https://img.shields.io/badge/TSDB-InfluxDB%20%2B%20PostgreSQL-22ADF6.svg)
![Security](https://img.shields.io/badge/Security-Sanitized%20%2F%20Zero--Leak-success.svg)

> [!IMPORTANT]
> **Confidentiality, Sanitization & NDA Notice:**  
> This repository represents a fully abstracted reference architecture synthesized from real-world telemetry infrastructure engineering. 
> All proprietary employer assets, company-specific network topologies, private hostnames, and credentials have been strictly purged. 
> The commit history has been intentionally wiped clean and initialized from scratch to guarantee zero secret retention and full NDA compliance.

---

## 1. Architectural Blueprint

```mermaid
graph TD
    subgraph Ingestion ["Edge & Ingestion Layer"]
        GW["LoRaWAN Gateways / Weather Stations"] -->|"Semtech Packet Forwarder<br/>(:1700 UDP)"| CS_GB["ChirpStack Gateway Bridge"]
        SENSORS["MQTT Sensors / Actuators"] -->|"TCP :1883"| MQTT["Eclipse Mosquitto MQTT Broker"]
    end

    subgraph CorePlatform ["Core Microservices & State (Kubernetes)"]
        CS_GB -->|"MQTT Topics"| MQTT
        MQTT -->|"Event Bus"| CS_NS["ChirpStack Network Server v4"]
        CS_NS -->|"Device Sessions / State"| REDIS[("Redis 7 (In-Memory)")]
        CS_NS -->|"Metadata / User Profiles"| PG[("PostgreSQL 15 (HA)")]
    end

    subgraph TelemetryPipeline ["Processing & Storage Pipeline"]
        MQTT -->|"Raw Data Streams"| TELEGRAF["Telegraf Aggregator"]
        TELEGRAF -->|"Batched Metrics"| INFLUX[("InfluxDB 2 (Time-Series TSDB)")]
    end

    subgraph Visualization ["Observability & Edge Access"]
        INFLUX --> GRAFANA["Grafana Dashboards"]
        PG --> GRAFANA
        INGRESS["Ingress-Nginx (TLS Termination)"] --> CS_NS
        INGRESS --> GRAFANA
    end
```

---

## 2. Platform Capabilities

* **Declarative Helm Packaging:** All microservices (ChirpStack, Mosquitto, InfluxDB, Telegraf, PostgreSQL) packaged into modular Helm templates.
* **Zero-Trust Secret Management:** Completely decoupled from static configuration; integrates with HashiCorp Vault / ExternalSecrets Operator.
* **High-Throughput Telemetry Ingestion:** Multi-protocol support (MQTT, LoRaWAN Gateway Bridge, UDP telemetry).
* **Time-Series Persistence:** Dual-engine storage utilizing PostgreSQL for relational entities and InfluxDB for high-cadence sensor metrics.

---

## 3. Deployment Guide

### Prerequisites
* Kubernetes cluster `1.28+`
* Helm `v3.12+`
* Ingress controller (e.g. `ingress-nginx`)
* Storage provisioner (e.g. `local-path` or CSI driver)

### Quick Start
```bash
# 1. Create target namespace
kubectl create namespace telemetry-system

# 2. Populate environment secrets (from sanitized template)
kubectl apply -f charts/iot-telemetry-platform/secrets.example.yaml

# 3. Deploy the platform via Helm
helm upgrade --install telemetry-platform ./charts/iot-telemetry-platform \
  --namespace telemetry-system \
  --values ./charts/iot-telemetry-platform/values.yaml

# 4. Verify deployment health
kubectl get pods,svc -n telemetry-system
```

---

## 4. Repository Structure

```text
├── charts/
│   └── iot-telemetry-platform/
│       ├── Chart.yaml              # Helm chart metadata
│       ├── values.yaml             # Production parameter definitions
│       ├── secrets.example.yaml    # Safe secret template (no hardcoded passwords)
│       └── templates/
│           ├── _helpers.tpl        # Template labels and naming conventions
│           ├── chirpstack.yaml     # LoRaWAN server deployments & services
│           ├── mosquitto.yaml      # MQTT broker cluster service
│           ├── postgresql.yaml     # StatefulSet with persistent storage
│           ├── redis.yaml          # In-memory cache deployment
│           ├── influxdb.yaml       # Time-series database StatefulSet
│           └── ingress.yaml        # TLS-enabled Ingress definitions
└── README.md                       # Architecture specification & security audit
```

---

## 5. Security & Maintenance
Maintained by **Temirbek Rakhimgalyiev** ([LinkedIn](https://www.linkedin.com/in/temirbek-rakhimgalyiev-443174246/)).  
All manifests adhere to CIS Kubernetes benchmarks and non-root execution policies.
