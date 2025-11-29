# Grafana

Grafana provides visualization and dashboarding for VMStation.

## Overview

| Property | Value |
|----------|-------|
| Port | 3000 (internal), 30300 (NodePort) |
| Storage | 2Gi persistent volume |
| Auth | Anonymous access (Viewer role) |
| URL | http://192.168.4.63:30300 |

## Configuration

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
```

### Authentication

```ini
[auth.anonymous]
enabled = true
org_name = Main Org.
org_role = Viewer

[auth]
disable_login_form = false
```

### Datasources

Pre-configured datasources:
- Prometheus: `http://prometheus:9090`
- Loki: `http://loki:3100`

## Access

### Web UI

Open http://192.168.4.63:30300

Default credentials: admin / admin

### Health Check

```bash
curl http://192.168.4.63:30300/api/health
# Returns: {"database": "ok"}
```

### API Access

```bash
# List datasources
curl -s http://192.168.4.63:30300/api/datasources | jq

# List dashboards
curl -s http://192.168.4.63:30300/api/search | jq
```

## Dashboards

### Pre-configured

| Dashboard | Purpose |
|-----------|---------|
| Kubernetes Cluster | Cluster overview |
| Node Exporter Full | System metrics |
| Loki Logs & Aggregation | Log viewing |
| Syslog Infrastructure | Syslog analysis |
| CoreDNS Performance | DNS metrics |

### Import Dashboard

1. Open Grafana
2. Click + → Import
3. Enter dashboard ID or paste JSON
4. Select datasource
5. Click Import

### Export Dashboard

```bash
# Get dashboard by UID
curl -s http://192.168.4.63:30300/api/dashboards/uid/<uid> | jq '.dashboard' > dashboard.json
```

## Troubleshooting

### Plugin Download Failed

```bash
kubectl set env deployment/grafana -n monitoring GF_INSTALL_PLUGINS-
kubectl rollout restart deployment/grafana -n monitoring
```

### Datasource Connection Failed

Check DNS resolution:
```bash
kubectl exec -n monitoring deployment/grafana -- nslookup prometheus.monitoring.svc.cluster.local
```

### Template Error

Ensure dashboard JSON has templating section:
```json
{
  "templating": {
    "list": []
  }
}
```

### Reset Admin Password

```bash
kubectl exec -n monitoring deployment/grafana -- grafana-cli admin reset-admin-password newpassword
```

## Related Documentation

- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
- [Dashboards](dashboards.md)
- [Grafana Fixes](../../troubleshooting/grafana-fixes.md)
