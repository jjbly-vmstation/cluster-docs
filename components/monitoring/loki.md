# Loki

Loki provides log aggregation for VMStation.

## Overview

| Property | Value |
|----------|-------|
| Port | 3100 (internal), 31100 (NodePort) |
| Storage | 20Gi persistent volume |
| Retention | 31 days |
| URL | http://192.168.4.63:31100 |

## Configuration

### Deployment

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: loki
  namespace: monitoring
spec:
  serviceName: loki
  replicas: 1
  template:
    spec:
      securityContext:
        runAsUser: 10001
        runAsGroup: 10001
        fsGroup: 10001
```

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
        period: 24h  # Must be 24h for boltdb-shipper
```

### Storage Config

```yaml
storage_config:
  boltdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache
    shared_store: filesystem
  filesystem:
    directory: /loki/chunks
```

### Retention

```yaml
limits_config:
  retention_period: 744h  # 31 days
```

## Access

### Ready Check

```bash
curl http://192.168.4.63:31100/ready
# Returns: ready
```

### List Labels

```bash
curl -s http://192.168.4.63:31100/loki/api/v1/labels | jq
```

### Query Logs

```bash
curl -s 'http://192.168.4.63:31100/loki/api/v1/query_range?query={namespace="monitoring"}&limit=100'
```

## LogQL Queries

### Basic Queries

```logql
# All logs from namespace
{namespace="monitoring"}

# Specific pod
{namespace="monitoring", pod=~"prometheus.*"}

# Error logs
{namespace="monitoring"} |= "error"
```

### Parse JSON

```logql
{namespace="monitoring"} | json | level="error"
```

### Aggregations

```logql
# Count by level
sum by (level) (count_over_time({namespace="monitoring"} | json [5m]))

# Rate of errors
rate({namespace="monitoring"} |= "error" [5m])
```

## Promtail Integration

Promtail ships logs to Loki.

### Promtail Configuration

```yaml
server:
  http_listen_port: 9080

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
```

### Labels Added

- namespace
- pod
- container
- node

## Troubleshooting

### Not Ready

Check logs:
```bash
kubectl logs loki-0 -n monitoring
```

Common fix - disable frontend_worker:
```yaml
# Comment out in loki config
# frontend_worker:
#   frontend_address: 127.0.0.1:9095
```

### Permission Issues

```bash
sudo chown -R 10001:10001 /srv/monitoring_data/loki
kubectl delete pod loki-0 -n monitoring
```

### Schema Error

Use 24h period for boltdb-shipper:
```yaml
schema_config:
  configs:
    - period: 24h  # Not 168h
```

## Related Documentation

- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
- [Monitoring Issues](../../troubleshooting/monitoring-issues.md)
- [Prometheus](prometheus.md)
