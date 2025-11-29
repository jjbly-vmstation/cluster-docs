# Prometheus

Prometheus is the metrics collection and alerting system for VMStation.

## Overview

| Property | Value |
|----------|-------|
| Port | 9090 (internal), 30090 (NodePort) |
| Storage | 10Gi persistent volume |
| Retention | 15 days / 8GB |
| URL | http://192.168.4.63:30090 |

## Configuration

### Deployment

Prometheus runs as a StatefulSet for persistent storage.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: prometheus
  namespace: monitoring
spec:
  serviceName: prometheus
  replicas: 1
  template:
    spec:
      securityContext:
        runAsUser: 65534
        runAsGroup: 65534
        fsGroup: 65534
```

### Storage

```yaml
volumeClaimTemplates:
- metadata:
    name: prometheus-storage
  spec:
    accessModes: ["ReadWriteOnce"]
    storageClassName: local-path
    resources:
      requests:
        storage: 10Gi
```

### Retention

```yaml
args:
  - '--storage.tsdb.retention.time=15d'
  - '--storage.tsdb.retention.size=8GB'
  - '--config.file=/etc/prometheus/prometheus.yml'
```

## Scrape Targets

### Kubernetes Nodes

```yaml
scrape_configs:
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
```

### Node Exporter

```yaml
  - job_name: 'node-exporter'
    static_configs:
      - targets:
          - '192.168.4.63:9100'
          - '192.168.4.61:9100'
          - '192.168.4.62:9100'
```

### Kubernetes Pods

```yaml
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

## Access

### Health Check

```bash
curl http://192.168.4.63:30090/-/healthy
# Returns: Prometheus is Healthy.
```

### Query Metrics

```bash
# Simple query
curl 'http://192.168.4.63:30090/api/v1/query?query=up'

# Range query
curl 'http://192.168.4.63:30090/api/v1/query_range?query=up&start=2025-01-01T00:00:00Z&end=2025-01-01T01:00:00Z&step=1m'
```

### Check Targets

```bash
curl -s http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'
```

## Common Queries

### CPU Usage

```promql
1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (node)
```

### Memory Usage

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes
```

### Disk Usage

```promql
1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)
```

### Pod Status

```promql
kube_pod_status_phase{phase="Running"}
```

## Troubleshooting

### CrashLoopBackOff

Check SecurityContext:
```yaml
securityContext:
  runAsUser: 65534
  runAsGroup: 65534  # Must be set
  fsGroup: 65534
```

### Permission Denied

```bash
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus
kubectl delete pod prometheus-0 -n monitoring
```

### TSDB Corruption

Prometheus auto-repairs. For severe cases:
```bash
kubectl scale statefulset prometheus -n monitoring --replicas=0
sudo rm -rf /srv/monitoring_data/prometheus/*
kubectl scale statefulset prometheus -n monitoring --replicas=1
```

## Related Documentation

- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
- [Grafana](grafana.md)
- [Exporters](exporters.md)
