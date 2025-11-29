# Monitoring Architecture

VMStation uses a comprehensive observability stack for metrics, logs, and visualization.

## Stack Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    Monitoring Stack                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     Grafana                               │   │
│  │                 (Visualization)                           │   │
│  │              http://192.168.4.63:30300                    │   │
│  └────────────────┬────────────────────┬─────────────────────┘   │
│                   │                    │                         │
│          ┌────────▼────────┐  ┌────────▼────────┐               │
│          │   Prometheus    │  │      Loki       │               │
│          │   (Metrics)     │  │    (Logs)       │               │
│          │   :30090        │  │    :31100       │               │
│          └────────▲────────┘  └────────▲────────┘               │
│                   │                    │                         │
│      ┌────────────┴────────────────────┴────────────┐           │
│      │                                               │           │
│  ┌───┴───┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │           │
│  │ Node  │ │ Blackbox │ │  IPMI    │ │kube-state│  │           │
│  │Exporter│ │ Exporter │ │ Exporter │ │ metrics  │  │           │
│  └───────┘ └──────────┘ └──────────┘ └──────────┘  │           │
│                                                     │           │
│      ┌──────────────────────────────────────────┐  │           │
│      │              Promtail                     │  │           │
│      │           (Log Shipper)                   │──┘           │
│      └──────────────────────────────────────────┘               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Components

### Prometheus

**Purpose**: Time-series metrics collection and alerting

**Deployment**: StatefulSet with persistent storage

```yaml
Port: 9090 (ClusterIP), 30090 (NodePort)
Storage: 10Gi persistent volume
Retention: 15 days / 8GB
```

**Scrape Targets**:
- Node Exporter (all nodes)
- Kubelet metrics
- kube-state-metrics
- Blackbox Exporter
- IPMI Exporter (optional)

### Grafana

**Purpose**: Visualization and dashboards

**Deployment**: Deployment with persistent storage

```yaml
Port: 3000 (ClusterIP), 30300 (NodePort)
Storage: 2Gi persistent volume
Auth: Anonymous access (Viewer role)
```

**Pre-configured Datasources**:
- Prometheus
- Loki

**Dashboards**:
- Kubernetes Cluster Overview
- Node Exporter Full
- Loki Logs & Aggregation
- Syslog Infrastructure
- CoreDNS Performance

### Loki

**Purpose**: Log aggregation and querying

**Deployment**: StatefulSet with persistent storage

```yaml
Port: 3100 (ClusterIP), 31100 (NodePort)
Storage: 20Gi persistent volume
Retention: 31 days
```

**Features**:
- Log indexing
- Label-based queries
- Integration with Grafana

### Promtail

**Purpose**: Log collection and shipping to Loki

**Deployment**: DaemonSet on all nodes

```yaml
Collects from:
- /var/log/containers/*.log
- /var/log/pods/*/*.log
```

**Labels added**:
- namespace
- pod
- container
- node

### Node Exporter

**Purpose**: Host-level metrics (CPU, memory, disk, network)

**Deployment**: DaemonSet on all nodes

```yaml
Port: 9100
Metrics: ~1000 per node
```

**Key Metrics**:
- node_cpu_seconds_total
- node_memory_MemAvailable_bytes
- node_filesystem_avail_bytes
- node_network_receive_bytes_total

### Blackbox Exporter

**Purpose**: Probe endpoints for availability

**Deployment**: Deployment on control plane

```yaml
Port: 9115
Probes: HTTP, TCP, DNS, ICMP
```

**Configured Probes**:
- HTTP endpoints
- DNS resolution
- Service health

### Kube-state-metrics

**Purpose**: Kubernetes object state metrics

**Deployment**: Deployment on control plane

```yaml
Port: 8080 (metrics), 8081 (telemetry)
```

**Key Metrics**:
- kube_pod_status_phase
- kube_deployment_status_replicas
- kube_node_status_condition

### IPMI Exporter (Optional)

**Purpose**: Hardware monitoring via IPMI/BMC

**Deployment**: Deployment on nodes with IPMI

```yaml
Port: 9290
Requires: IPMI credentials
```

**Metrics**:
- Temperature sensors
- Fan speeds
- Power consumption

## Data Flow

### Metrics Flow

```
┌─────────────────────────────────────────────────────────────┐
│                       Metrics Flow                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Node Exporter ────┐                                        │
│  (system metrics)  │                                        │
│                    │                                        │
│  kube-state ───────┼──▶ Prometheus ──▶ Grafana Dashboards  │
│  (k8s objects)     │     (scrapes      (visualizes         │
│                    │      every 15s)    queries)            │
│  Blackbox ─────────┘                                        │
│  (probes)                                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Logs Flow

```
┌─────────────────────────────────────────────────────────────┐
│                        Logs Flow                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Container logs ───┐                                        │
│  (/var/log/pods)   │                                        │
│                    │                                        │
│  System logs ──────┼──▶ Promtail ──▶ Loki ──▶ Grafana      │
│  (/var/log/*)      │    (ships     (indexes   (queries      │
│                    │     logs)      & stores)  & displays)  │
│  Journal logs ─────┘                                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Prometheus Configuration

### Scrape Config Example

```yaml
scrape_configs:
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - source_labels: [__address__]
        regex: (.+):10250
        target_label: __address__
        replacement: ${1}:9100

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

### Recording Rules

```yaml
groups:
  - name: node
    rules:
      - record: node:cpu:usage
        expr: 1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (node)
```

### Alert Rules

```yaml
groups:
  - name: alerts
    rules:
      - alert: NodeDown
        expr: up{job="node-exporter"} == 0
        for: 5m
        labels:
          severity: critical
```

## Loki Configuration

### Schema Config

```yaml
schema_config:
  configs:
    - from: 2020-01-01
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: loki_index_
        period: 24h
```

### Query Examples

```logql
# All logs from monitoring namespace
{namespace="monitoring"}

# Error logs
{namespace="monitoring"} |= "error"

# Parse JSON logs
{namespace="monitoring"} | json | level="error"
```

## Access and URLs

| Service | Internal | External | Purpose |
|---------|----------|----------|---------|
| Prometheus | prometheus.monitoring:9090 | 192.168.4.63:30090 | Metrics |
| Grafana | grafana.monitoring:3000 | 192.168.4.63:30300 | Dashboards |
| Loki | loki.monitoring:3100 | 192.168.4.63:31100 | Logs |

## Health Checks

### Prometheus Health

```bash
curl http://192.168.4.63:30090/-/healthy
# Returns: Prometheus is Healthy.
```

### Loki Ready

```bash
curl http://192.168.4.63:31100/ready
# Returns: ready
```

### Grafana Health

```bash
curl http://192.168.4.63:30300/api/health
# Returns: {"database": "ok"}
```

## Troubleshooting

### Check Targets

```bash
# Prometheus targets
curl -s http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'
```

### Check Datasources

```bash
# Grafana datasources
curl -s http://192.168.4.63:30300/api/datasources | jq '.[].name'
```

### Validate Logs

```bash
# Check Loki ingestion
curl -s http://192.168.4.63:31100/loki/api/v1/labels | jq
```

## Related Documentation

- [Architecture Overview](overview.md)
- [Storage Architecture](storage-architecture.md)
- [Prometheus Component](../components/monitoring/prometheus.md)
- [Grafana Component](../components/monitoring/grafana.md)
- [Loki Component](../components/monitoring/loki.md)
