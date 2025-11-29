# Monitoring Architecture

Observability stack design for VMStation.

## Monitoring Overview

```
┌──────────────────────────────────────────────────────────┐
│                    Monitoring Stack                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │   Grafana   │◄───│ Prometheus  │◄───│Node Exporter│  │
│  │   :30300    │    │   :30090    │    │   :9100     │  │
│  └──────┬──────┘    └─────────────┘    └─────────────┘  │
│         │                                                │
│         │           ┌─────────────┐    ┌─────────────┐  │
│         └──────────►│    Loki     │◄───│  Promtail   │  │
│                     │   :31100    │    │             │  │
│                     └─────────────┘    └─────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Components

### Prometheus

Metrics collection and alerting.

| Property | Value |
|----------|-------|
| Port | 9090 (internal), 30090 (NodePort) |
| Retention | 15 days |
| Storage | 10Gi PVC |

**Scrape Targets:**
- Node Exporter (all nodes)
- kube-state-metrics
- Blackbox Exporter
- IPMI Exporter (optional)

### Grafana

Dashboards and visualization.

| Property | Value |
|----------|-------|
| Port | 3000 (internal), 30300 (NodePort) |
| Storage | 2Gi PVC |
| Auth | Anonymous (Viewer) enabled |

**Datasources:**
- Prometheus
- Loki

### Loki

Log aggregation system.

| Property | Value |
|----------|-------|
| Port | 3100 (internal), 31100 (NodePort) |
| Retention | 31 days |
| Storage | 20Gi PVC |

### Promtail

Log shipping agent (DaemonSet).

| Property | Value |
|----------|-------|
| Deployment | DaemonSet (all nodes) |
| Sources | /var/log/containers/*.log |

### Node Exporter

System metrics exporter (DaemonSet).

| Property | Value |
|----------|-------|
| Port | 9100 |
| Deployment | DaemonSet (all nodes) |

### Blackbox Exporter

Probe endpoints for availability.

| Property | Value |
|----------|-------|
| Port | 9115 |
| Probes | HTTP, TCP, DNS |

## Data Flow

### Metrics Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│Node Exporter│────►│ Prometheus  │────►│   Grafana   │
│   (9100)    │     │   (9090)    │     │   (3000)    │
└─────────────┘     └─────────────┘     └─────────────┘
```

### Logs Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Promtail   │────►│    Loki     │────►│   Grafana   │
│ (DaemonSet) │     │   (3100)    │     │   (3000)    │
└─────────────┘     └─────────────┘     └─────────────┘
```

## Service URLs

| Service | Internal URL | External URL |
|---------|--------------|--------------|
| Prometheus | prometheus:9090 | http://192.168.4.63:30090 |
| Grafana | grafana:3000 | http://192.168.4.63:30300 |
| Loki | loki:3100 | http://192.168.4.63:31100 |

## Dashboards

### Pre-configured Dashboards

- **Kubernetes Cluster** - Node and pod overview
- **Node Exporter** - System metrics
- **Loki Logs** - Log aggregation
- **Syslog Infrastructure** - Syslog monitoring
- **CoreDNS** - DNS performance

### Dashboard Location

Dashboards provisioned via ConfigMap:
```
manifests/monitoring/grafana.yaml
ansible/files/grafana_dashboards/
```

## Alerting

### Prometheus Alerting Rules

Located in Prometheus ConfigMap:
```yaml
groups:
- name: node
  rules:
  - alert: NodeDown
    expr: up{job="node-exporter"} == 0
    for: 5m
```

### Alert Destinations

Currently configured for local Grafana alerts. Future: AlertManager integration.

## Security Context

### Prometheus

```yaml
securityContext:
  runAsUser: 65534
  runAsGroup: 65534
  fsGroup: 65534
  runAsNonRoot: true
```

### Loki

```yaml
securityContext:
  runAsUser: 10001
  runAsGroup: 10001
  fsGroup: 10001
```

### Grafana

```yaml
securityContext:
  runAsUser: 472
  fsGroup: 472
```

## High Availability

Current deployment is single-instance. For HA:

- Multiple Prometheus replicas with Thanos
- Loki in microservices mode
- Multiple Grafana instances with shared database

## Troubleshooting

### Check Pod Status

```bash
kubectl get pods -n monitoring
```

### Check Endpoints

```bash
kubectl get endpoints -n monitoring
```

### Validate Health

```bash
curl http://192.168.4.63:30090/-/healthy  # Prometheus
curl http://192.168.4.63:31100/ready      # Loki
curl http://192.168.4.63:30300/api/health # Grafana
```

## Related Documentation

- [Architecture Overview](overview.md)
- [Monitoring Deployment](../deployment/monitoring-deployment.md)
- [Monitoring Issues](../troubleshooting/monitoring-issues.md)
- [Prometheus](../components/monitoring/prometheus.md)
- [Grafana](../components/monitoring/grafana.md)
- [Loki](../components/monitoring/loki.md)
