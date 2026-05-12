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
├── prometheus/
│   ├── prometheus.yml     # Scrape configuration
│   └── alert-rules.yml    # Alerting rules
├── grafana/
│   ├── dashboards/        # Dashboard JSON files
│   └── provisioning/      # Datasource and dashboard provisioning
├── loki/                  # Loki log aggregation config
├── promtail/              # Promtail log collector config
├── alertmanager/          # Alertmanager routing config
├── exporters/             # Custom exporter configs
├── diagrams/              # Architecture diagrams
└── docker-compose.yml     # Stack orchestration
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

2. Copy the environment file and configure:

```bash
cp .env.example .env
```

Edit `.env` and set your values:
- `GF_SECURITY_ADMIN_PASSWORD` — Grafana admin password
- `TEAMS_WEBHOOK_URL` — Microsoft Teams incoming webhook URL

3. Start the monitoring stack:

```bash
docker compose up -d
```

4. Verify services are running:

```bash
docker compose ps
```

## Service Endpoints

| Service       | URL                    | Description                |
|---------------|------------------------|----------------------------|
| Prometheus    | http://localhost:9090   | Metrics & query engine     |
| Grafana       | http://localhost:3000   | Dashboards & visualization |
| Alertmanager  | http://localhost:9093   | Alert management UI        |
| Loki          | http://localhost:3100   | Log aggregation API        |
| Node Exporter | http://localhost:9100   | Host metrics (internal)    |

## Grafana

Default credentials:

- **Username:** `admin`
- **Password:** `admin` (change via `GF_SECURITY_ADMIN_PASSWORD` in `.env`)

### Provisioned Dashboards

- **Linux System Dashboard** — CPU, memory, disk I/O, network, and filesystem metrics from Node Exporter

Dashboards are auto-provisioned on startup via the `grafana/provisioning/` directory. To add new dashboards, place JSON files in `grafana/dashboards/` and they will load automatically.

### Datasources

Both Prometheus and Loki are auto-provisioned as Grafana datasources. No manual configuration needed after startup.

## Centralized Logging

The stack uses **Loki** for log aggregation and **Promtail** as the log collector.

### How It Works

1. **Promtail** discovers running Docker containers via the Docker socket
2. Container logs are scraped from `/var/lib/docker/containers/`
3. System logs are collected from `/var/log/*.log`
4. All logs are pushed to **Loki** for indexing and querying
5. **Grafana** queries Loki to visualize and search logs

### Querying Logs in Grafana

Navigate to **Explore** in Grafana, select the **Loki** datasource, and use LogQL:

```logql
{job="docker"} |= "error"
{container="prometheus"} | logfmt | level="error"
```

## Alerting

Alerts are managed through **Prometheus** alert rules and routed via **Alertmanager** to Microsoft Teams.

### Configured Alert Rules

| Alert            | Condition                | Severity | Duration |
|------------------|--------------------------|----------|----------|
| HighCPUUsage     | CPU usage > 80%          | warning  | 5m       |
| HighMemoryUsage  | Memory usage > 85%       | warning  | 5m       |
| HostDown         | Target unreachable       | critical | 2m       |

### Microsoft Teams Integration

1. Create an **Incoming Webhook** connector in your Teams channel
2. Copy the webhook URL
3. Set it in your `.env` file:

```
TEAMS_WEBHOOK_URL=https://outlook.office.com/webhook/your-actual-url
```

4. Restart the stack:

```bash
docker compose restart alertmanager
```

Alerts will be sent to your Teams channel when triggered and when resolved.

### Checking Alert Status

- **Prometheus Alerts:** http://localhost:9090/alerts
- **Alertmanager UI:** http://localhost:9093

## Architecture

> Coming soon — architecture diagram will be added.
