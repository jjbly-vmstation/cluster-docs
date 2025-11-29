# Dashboards

Grafana dashboard configuration.

## Pre-configured Dashboards

### Kubernetes Cluster Overview

Overview of cluster resources:
- Node status
- Pod count
- CPU/memory usage
- Network I/O

### Node Exporter Full

Detailed system metrics:
- CPU breakdown
- Memory usage
- Disk I/O
- Network statistics
- System load

### Loki Logs & Aggregation

Log analysis:
- Log volume
- Error rates
- Top log sources
- Live tail

### Syslog Infrastructure

Syslog monitoring:
- Message rates
- Severity breakdown
- Facility filtering
- Recent events

### CoreDNS Performance

DNS metrics:
- Query rate
- Response times (P50, P95, P99)
- Cache hit rate
- Error rates

## Dashboard Location

### Provisioning

Dashboards are provisioned via ConfigMap:

```
manifests/monitoring/grafana.yaml
```

### JSON Files

Dashboard JSON files:

```
ansible/files/grafana_dashboards/
├── kubernetes-dashboard.json
├── node-exporter-dashboard.json
├── loki-dashboard.json
├── syslog-dashboard.json
└── coredns-dashboard.json
```

## Creating Dashboards

### Via UI

1. Create → Dashboard
2. Add panels
3. Configure queries
4. Save dashboard

### Dashboard JSON

Required structure:

```json
{
  "schemaVersion": 27,
  "title": "My Dashboard",
  "uid": "unique-id",
  "templating": {
    "list": []  // Required even if empty
  },
  "panels": [
    {
      "id": 1,
      "type": "timeseries",
      "title": "Panel Title",
      "targets": [
        {
          "expr": "up",
          "datasource": "Prometheus"
        }
      ]
    }
  ]
}
```

### Template Variables

Add template variables for filtering:

```json
{
  "templating": {
    "list": [
      {
        "name": "namespace",
        "type": "query",
        "query": "label_values(kube_pod_info, namespace)",
        "datasource": "Prometheus"
      }
    ]
  }
}
```

Use in queries: `$namespace`

## Importing Dashboards

### From Grafana.com

1. Dashboards → Import
2. Enter dashboard ID
3. Select datasource
4. Import

**Popular IDs:**
- 1860 - Node Exporter Full
- 7249 - Kubernetes Cluster
- 13639 - Loki

### From JSON

1. Dashboards → Import
2. Upload JSON file
3. Configure
4. Import

## Exporting Dashboards

### Single Dashboard

1. Open dashboard
2. Settings → JSON Model
3. Copy JSON

### All Dashboards

```bash
for uid in $(curl -s http://192.168.4.63:30300/api/search | jq -r '.[].uid'); do
  curl -s "http://192.168.4.63:30300/api/dashboards/uid/$uid" > "dashboard-$uid.json"
done
```

## Troubleshooting

### Template Variable Error

```
Error updating options: map is not a function
```

**Cause:** Missing `templating` section

**Fix:** Add to dashboard JSON:
```json
{
  "templating": {
    "list": []
  }
}
```

### No Data

1. Check datasource connection
2. Verify query syntax
3. Check time range
4. Verify data exists

### Dashboard Not Loading

1. Check Grafana pod status
2. Check ConfigMap
3. Review Grafana logs

## Best Practices

### Organization

- Use folders for related dashboards
- Consistent naming convention
- Tag dashboards appropriately

### Variables

- Use template variables for flexibility
- Add default values
- Include "All" option

### Panels

- Clear titles
- Appropriate visualization type
- Meaningful colors
- Include units

### Performance

- Limit query range
- Use aggregations
- Avoid too many panels

## Related Documentation

- [Grafana](grafana.md)
- [Prometheus](prometheus.md)
- [Loki](loki.md)
