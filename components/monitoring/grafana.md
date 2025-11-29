# Grafana

Visualization and dashboarding platform.

## Overview

Grafana provides dashboards for metrics and logs visualization.

| Property | Value |
|----------|-------|
| Version | 10.x |
| Port | 3000 (internal), 30300 (NodePort) |
| Storage | 2Gi PVC |
| Auth | Anonymous (Viewer) enabled |

## Access

- URL: http://192.168.4.63:30300
- Default credentials: admin/admin (change on first login)

## Datasources

### Pre-configured

| Name | Type | URL |
|------|------|-----|
| Prometheus | prometheus | http://prometheus:9090 |
| Loki | loki | http://loki:3100 |

### Add Datasource

1. Configuration → Data Sources
2. Add data source
3. Select type
4. Configure URL
5. Save & Test

## Dashboards

### Pre-configured Dashboards

- **Kubernetes Cluster** - Node and pod overview
- **Node Exporter Full** - Detailed system metrics
- **Loki Logs** - Log aggregation
- **Syslog Infrastructure** - Syslog monitoring
- **CoreDNS Performance** - DNS metrics

### Import Dashboard

1. Dashboards → Import
2. Enter dashboard ID or upload JSON
3. Select datasource
4. Import

Popular dashboard IDs:
- 1860 - Node Exporter Full
- 7249 - Kubernetes Cluster
- 13639 - Loki Dashboard

### Export Dashboard

1. Go to Dashboard
2. Settings → JSON Model
3. Copy JSON

## Configuration

### Anonymous Access

```yaml
env:
  - name: GF_AUTH_ANONYMOUS_ENABLED
    value: "true"
  - name: GF_AUTH_ANONYMOUS_ORG_ROLE
    value: "Viewer"
```

### Admin Password

```yaml
env:
  - name: GF_SECURITY_ADMIN_PASSWORD
    value: "newpassword"
```

### Plugins

Install plugins via environment variable:

```yaml
env:
  - name: GF_INSTALL_PLUGINS
    value: "grafana-clock-panel,grafana-piechart-panel"
```

**Note:** Plugin installation may fail if network unavailable.

## Alerting

### Create Alert

1. Edit panel
2. Alert tab
3. Create alert rule
4. Set conditions
5. Configure notifications

### Notification Channels

1. Alerting → Notification channels
2. Add channel
3. Configure (email, Slack, webhook)
4. Test

## Provisioning

### Datasources

```yaml
# /etc/grafana/provisioning/datasources/datasources.yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true
```

### Dashboards

```yaml
# /etc/grafana/provisioning/dashboards/dashboards.yaml
apiVersion: 1
providers:
  - name: default
    folder: ''
    type: file
    options:
      path: /var/lib/grafana/dashboards
```

## Troubleshooting

### Cannot Connect

```bash
# Check pod
kubectl get pods -n monitoring -l app=grafana
kubectl logs -n monitoring -l app=grafana

# Test endpoint
curl http://192.168.4.63:30300/api/health
```

### Datasource Not Working

```bash
# Test from Grafana pod
kubectl exec -n monitoring -l app=grafana -- wget -O- http://prometheus:9090/-/healthy
```

### Plugin Failed

Remove plugin environment variable:

```bash
kubectl set env deployment/grafana -n monitoring GF_INSTALL_PLUGINS-
```

### Dashboard Error

Check for missing `templating` section:

```json
{
  "templating": {
    "list": []
  }
}
```

## Backup

### Export All Dashboards

```bash
for uid in $(curl -s http://192.168.4.63:30300/api/search | jq -r '.[].uid'); do
  curl -s "http://192.168.4.63:30300/api/dashboards/uid/$uid" > "dashboard-$uid.json"
done
```

### Backup Database

```bash
tar -czf grafana-backup.tar.gz /srv/monitoring_data/*grafana*/
```

## Related Documentation

- [Dashboards](dashboards.md)
- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
- [Grafana Fixes](../../troubleshooting/grafana-fixes.md)
