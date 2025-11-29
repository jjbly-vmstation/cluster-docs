# Loki

Log aggregation system.

## Overview

Loki collects and indexes logs from all cluster nodes.

| Property | Value |
|----------|-------|
| Version | 2.x |
| Port | 3100 (internal), 31100 (NodePort) |
| Storage | 20Gi PVC |
| Retention | 31 days |

## Access

- URL: http://192.168.4.63:31100
- Health: http://192.168.4.63:31100/ready

## Architecture

### Components (Single Instance)

In single-instance mode, Loki runs:
- Ingester
- Querier
- Query frontend (disabled for single-instance)
- Compactor

### Data Flow

```
Promtail → Loki (Push API) → Storage → Query API → Grafana
```

## Configuration

### Storage Schema

```yaml
schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h  # Must be 24h for boltdb-shipper
```

### Retention

```yaml
limits_config:
  retention_period: 744h  # 31 days
```

### Single Instance Mode

For single-instance, disable frontend_worker:

```yaml
# Comment out:
# frontend_worker:
#   frontend_address: 127.0.0.1:9095
```

## Querying

### LogQL Examples

```logql
# All logs from namespace
{namespace="monitoring"}

# Error logs
{namespace="monitoring"} |= "error"

# JSON parsing
{job="syslog"} | json

# Filter parsed fields
{job="syslog"} | json | severity="error"

# Ignore parse errors
{job="syslog"} | json | __error__=""

# Rate of logs
rate({namespace="monitoring"}[5m])
```

### Query via API

```bash
# Query logs
curl -G 'http://192.168.4.63:31100/loki/api/v1/query' \
  --data-urlencode 'query={namespace="monitoring"}'

# Get labels
curl http://192.168.4.63:31100/loki/api/v1/labels

# Get label values
curl http://192.168.4.63:31100/loki/api/v1/label/namespace/values
```

### Query in Grafana

1. Go to Explore
2. Select Loki datasource
3. Enter LogQL query
4. Run query

## Troubleshooting

### CrashLoopBackOff

**Check logs:**
```bash
kubectl logs loki-0 -n monitoring
```

**Common issues:**

1. **Connection refused (9095)**
   ```
   Error: dial tcp 127.0.0.1:9095: connection refused
   ```
   
   Fix: Disable frontend_worker in config.

2. **Schema validation error**
   ```
   Error: boltdb-shipper requires 24h index period
   ```
   
   Fix: Change period to 24h.

3. **Permission denied**
   ```bash
   sudo chown -R 10001:10001 /srv/monitoring_data/*loki*/
   ```

### No Logs Appearing

1. **Check Promtail:**
   ```bash
   kubectl get pods -n monitoring -l app=promtail
   kubectl logs -n monitoring -l app=promtail
   ```

2. **Check Loki is ready:**
   ```bash
   curl http://192.168.4.63:31100/ready
   ```

3. **Check ingestion:**
   ```bash
   curl http://192.168.4.63:31100/metrics | grep loki_ingester
   ```

### Query Timeout

Increase query timeout or reduce query range.

## Promtail

Log shipping agent that sends logs to Loki.

### Deployment

Runs as DaemonSet on all nodes.

### Configuration

```yaml
positions:
  filename: /var/log/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
```

### Troubleshooting Promtail

```bash
kubectl get pods -n monitoring -l app=promtail
kubectl logs -n monitoring <promtail-pod>
```

## Related Documentation

- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
- [Monitoring Deployment](../../deployment/monitoring-deployment.md)
- [Monitoring Issues](../../troubleshooting/monitoring-issues.md)
