# Prometheus

Metrics collection and alerting system.

## Overview

Prometheus collects and stores time-series metrics from the cluster.

| Property | Value |
|----------|-------|
| Version | 2.x |
| Port | 9090 (internal), 30090 (NodePort) |
| Storage | 10Gi PVC |
| Retention | 15 days |

## Access

- URL: http://192.168.4.63:30090
- Health: http://192.168.4.63:30090/-/healthy

## Configuration

### Deployment

Deployed as StatefulSet with 2 containers:
- `prometheus` - Main server
- `config-reloader` - Hot reload configuration

### SecurityContext

```yaml
securityContext:
  runAsUser: 65534
  runAsGroup: 65534
  fsGroup: 65534
  runAsNonRoot: true
```

### Retention

```yaml
args:
  - '--storage.tsdb.retention.time=15d'
  - '--storage.tsdb.retention.size=8GB'
  - '--storage.tsdb.wal-compression'
```

## Scrape Targets

### Configured Targets

| Target | Job | Port |
|--------|-----|------|
| Node Exporter | node-exporter | 9100 |
| Blackbox Exporter | blackbox | 9115 |
| Kube-state-metrics | kube-state-metrics | 8080 |
| IPMI Exporter | ipmi-exporter | 9290 |

### Check Targets

```bash
# Via API
curl http://192.168.4.63:30090/api/v1/targets | jq

# Via UI
# Navigate to Status → Targets
```

## Querying

### PromQL Examples

```promql
# CPU usage
100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk usage
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100

# Up status
up{job="node-exporter"}

# Request rate
rate(http_requests_total[5m])
```

### Query API

```bash
# Instant query
curl 'http://192.168.4.63:30090/api/v1/query?query=up'

# Range query
curl 'http://192.168.4.63:30090/api/v1/query_range?query=up&start=2024-01-01T00:00:00Z&end=2024-01-01T01:00:00Z&step=15s'
```

## Alerting

### Alert Rules

```yaml
groups:
- name: node
  rules:
  - alert: NodeDown
    expr: up{job="node-exporter"} == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Node {{ $labels.instance }} is down"
```

### Check Alerts

```bash
# Active alerts
curl http://192.168.4.63:30090/api/v1/alerts | jq

# Alert rules
curl http://192.168.4.63:30090/api/v1/rules | jq
```

## Troubleshooting

### Pod CrashLoopBackOff

**Check logs:**
```bash
kubectl logs prometheus-0 -n monitoring -c prometheus
```

**Common issues:**
1. Permission denied - Add `runAsGroup`
2. TSDB corruption - Will auto-recover
3. OOM - Increase memory limits

### Empty Endpoints

Prometheus endpoints empty = pods not Ready.
Fix pod issues first.

### Targets Down

```bash
# Check target status
curl http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job, instance, health}'
```

## Maintenance

### Reload Configuration

```bash
curl -X POST http://192.168.4.63:30090/-/reload
```

### Compact TSDB

```bash
curl -X POST http://192.168.4.63:30090/api/v1/admin/tsdb/compact
```

### Take Snapshot

```bash
curl -X POST http://192.168.4.63:30090/api/v1/admin/tsdb/snapshot
```

## Related Documentation

- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
- [Monitoring Deployment](../../deployment/monitoring-deployment.md)
- [Monitoring Issues](../../troubleshooting/monitoring-issues.md)
