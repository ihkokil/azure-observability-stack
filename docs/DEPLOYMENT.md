# Deployment Guide

Step-by-step guide for deploying the observability stack on a Linux host.

## Prerequisites

- **OS:** Ubuntu 20.04+ / Debian 11+ (or any Linux with Docker support)
- **Docker Engine:** 20.10 or later
- **Docker Compose:** v2.0 or later (included with Docker Desktop)
- **RAM:** Minimum 2GB free (4GB recommended for production)
- **Disk:** At least 10GB free for metric and log storage

### Install Docker (if not already installed)

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

Log out and back in for group changes to take effect.

## Deployment Steps

### 1. Clone the Repository

```bash
git clone https://github.com/ihkokil/azure-observability-stack.git
cd azure-observability-stack
```

### 2. Configure Environment

```bash
cp .env.example .env
```

Edit `.env` with your settings:

```bash
# Set a strong Grafana admin password
GF_SECURITY_ADMIN_PASSWORD=your-secure-password

# Set your Microsoft Teams webhook URL
TEAMS_WEBHOOK_URL=https://outlook.office.com/webhook/your-actual-webhook-url
```

### 3. Start the Stack

```bash
docker compose up -d
```

### 4. Verify All Services

```bash
docker compose ps
```

All containers should show `running` status. You can also check individual services:

```bash
# Check Prometheus targets
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'

# Check Loki readiness
curl -s http://localhost:3100/ready

# Check Alertmanager status
curl -s http://localhost:9093/api/v2/status | jq '.cluster.status'
```

### 5. Access Grafana

Open http://localhost:3000 in your browser.

- **Username:** `admin`
- **Password:** value from `GF_SECURITY_ADMIN_PASSWORD` in `.env`

Dashboards and datasources are automatically provisioned — no manual setup required.

## Updating the Stack

Pull latest images and restart:

```bash
docker compose pull
docker compose up -d
```

## Stopping the Stack

```bash
# Stop containers (preserves data)
docker compose stop

# Stop and remove containers (preserves volumes)
docker compose down

# Stop and remove everything including volumes (DATA LOSS)
docker compose down -v
```

## Troubleshooting

### Container won't start

Check the logs for the failing container:

```bash
docker compose logs <service-name>
# Example: docker compose logs prometheus
```

### Prometheus shows targets as DOWN

1. Verify the target container is running: `docker compose ps`
2. Check network connectivity:

```bash
docker exec prometheus wget -qO- http://node-exporter:9100/metrics | head
```

3. Validate prometheus config:

```bash
docker exec prometheus promtool check config /etc/prometheus/prometheus.yml
```

### Grafana shows "No data" on dashboards

1. Confirm Prometheus is receiving metrics: visit http://localhost:9090/targets
2. Check the datasource provisioning:

```bash
docker compose logs grafana | grep -i "provisioning"
```

3. Try querying directly in Grafana Explore with `up` metric

### Loki not receiving logs

1. Verify Promtail can access Docker socket:

```bash
docker exec promtail ls -la /var/run/docker.sock
```

2. Check Promtail targets:

```bash
curl -s http://localhost:9080/targets | head
```

3. Check Promtail logs:

```bash
docker compose logs promtail
```

### Alerts not firing

1. Check alert rule evaluation: http://localhost:9090/alerts
2. Verify Alertmanager is connected: http://localhost:9090/status (check "Alertmanagers" section)
3. Test webhook URL:

```bash
curl -X POST "$TEAMS_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"text": "Test alert from observability stack"}'
```

### High disk usage from logs/metrics

Clean up old data:

```bash
# Restart Loki to compact chunks
docker compose restart loki

# Prometheus automatically handles retention (default: 30d)
# To force cleanup, restart Prometheus
docker compose restart prometheus
```

## Resource Usage

Typical resource consumption on a single host:

| Service       | CPU (idle) | RAM (idle) | RAM (load) |
|---------------|-----------|-----------|-----------|
| Prometheus    | ~1%       | ~200MB    | ~500MB    |
| Grafana       | <1%       | ~100MB    | ~200MB    |
| Loki          | <1%       | ~100MB    | ~300MB    |
| Promtail      | <1%       | ~50MB     | ~100MB    |
| Alertmanager  | <1%       | ~30MB     | ~50MB     |
| Node Exporter | <1%       | ~20MB     | ~30MB     |
