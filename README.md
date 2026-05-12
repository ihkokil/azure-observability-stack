# Azure Observability Stack

Production-ready monitoring and observability stack for cloud infrastructure, built with Docker Compose.

## Tech Stack

- **Prometheus** — Metrics collection and alerting
- **Grafana** — Visualization and dashboards
- **Loki** — Log aggregation
- **Promtail** — Log shipping agent
- **Alertmanager** — Alert routing and notifications
- **Node Exporter** — Host-level metrics

## Project Structure

```
azure-observability-stack/
├── prometheus/          # Prometheus configuration
├── grafana/
│   ├── dashboards/      # Dashboard JSON files
│   └── provisioning/    # Datasource and dashboard provisioning
├── loki/                # Loki log aggregation config
├── promtail/            # Promtail log collector config
├── alertmanager/        # Alertmanager routing config
├── exporters/           # Custom exporter configs
├── diagrams/            # Architecture diagrams
└── docker-compose.yml   # Stack orchestration
```

## Prerequisites

- Docker Engine 20.10+
- Docker Compose v2.0+
- Minimum 2GB RAM available for the stack

## Getting Started

1. Clone the repository:

```bash
git clone https://github.com/your-username/azure-observability-stack.git
cd azure-observability-stack
```

2. Copy the environment file:

```bash
cp .env.example .env
```

3. Start the monitoring stack:

```bash
docker compose up -d
```

4. Verify services are running:

```bash
docker compose ps
```

## Service Endpoints

| Service       | URL                    | Description              |
|---------------|------------------------|--------------------------|
| Prometheus    | http://localhost:9090   | Metrics & query engine   |
| Grafana       | http://localhost:3000   | Dashboards & visualization |
| Node Exporter | http://localhost:9100   | Host metrics (internal)  |

## Grafana

Default credentials:

- **Username:** `admin`
- **Password:** `admin` (change via `GF_SECURITY_ADMIN_PASSWORD` in `.env`)

### Provisioned Dashboards

- **Linux System Dashboard** — CPU, memory, disk I/O, network, and filesystem metrics from Node Exporter

Dashboards are auto-provisioned on startup via the `grafana/provisioning/` directory. To add new dashboards, place JSON files in `grafana/dashboards/` and they will load automatically.

## Architecture

> Coming soon — architecture diagram will be added.
