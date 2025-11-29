# Dashboards

Pre-configured Grafana dashboards for VMStation.

## Available Dashboards

| Dashboard | Purpose | Datasource |
|-----------|---------|------------|
| Kubernetes Cluster | Cluster overview | Prometheus |
| Node Exporter Full | System metrics | Prometheus |
| Loki Logs & Aggregation | Log viewing | Loki |
| Syslog Infrastructure | Syslog analysis | Loki |
| CoreDNS Performance | DNS metrics | Prometheus |

## Kubernetes Cluster Dashboard

### Metrics Shown

- Node status
- Pod status by namespace
- CPU usage
- Memory usage
- Network I/O
- Disk I/O

### Key Panels

- Cluster Health Overview
- Node Resource Utilization
- Pod CPU/Memory
- Container Restart Count

## Node Exporter Dashboard

### Metrics Shown

- CPU (user, system, idle)
- Memory (used, cached, available)
- Disk (read/write, IOPS)
- Network (bytes, packets, errors)
- System (load, uptime)

### Variables

- `job`: Node exporter job
- `instance`: Target node

## Loki Logs Dashboard

### Features

- Live log tail
- Severity filtering
- Namespace filtering
- Top log producers
- Error rate tracking

### Variables

- `namespace`: Filter by namespace
- `severity`: Filter by log level

### Panels

- Service Health Indicators
- Log Ingestion Rate
- Error/Warning Rate
- Top Log-Producing Pods
- Log Stream

## Syslog Dashboard

### Metrics Shown

- Message rate
- Processing latency
- Severity distribution
- Facility breakdown
- Critical/Error logs

### Panels

- Syslog Message Rate
- Messages by Severity
- Messages by Facility
- Recent Critical Events
- Live Log Tail

## CoreDNS Dashboard

### Metrics Shown

- Query rate by type
- Response time (P50, P95, P99)
- Cache hit rate
- Forward requests
- Error rate

### Panels

- DNS Query Rate
- Response Latency Percentiles
- Cache Statistics
- Top Queried Domains
- Plugin Status

## Creating Custom Dashboards

### Template

```json
{
  "schemaVersion": 27,
  "title": "My Dashboard",
  "templating": {
    "list": []
  },
  "panels": [],
  "refresh": "30s"
}
```

### Add Variable

```json
{
  "templating": {
    "list": [
      {
        "name": "namespace",
        "type": "query",
        "datasource": "Prometheus",
        "query": "label_values(namespace)",
        "refresh": 1
      }
    ]
  }
}
```

### Add Panel

```json
{
  "panels": [
    {
      "title": "CPU Usage",
      "type": "graph",
      "datasource": "Prometheus",
      "targets": [
        {
          "expr": "1 - avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m]))"
        }
      ]
    }
  ]
}
```

## Import/Export

### Export Dashboard

```bash
curl -s http://192.168.4.63:30300/api/dashboards/uid/<uid> | jq '.dashboard' > dashboard.json
```

### Import Dashboard

1. Grafana UI → + → Import
2. Paste JSON or enter ID
3. Select datasource
4. Save

### From grafana.com

1. Find dashboard at grafana.com/dashboards
2. Copy dashboard ID
3. Import in Grafana UI

## Troubleshooting

### Template Error

Ensure templating section exists:
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

### Slow Loading

1. Reduce query range
2. Optimize PromQL
3. Add query caching

## Related Documentation

- [Grafana](grafana.md)
- [Prometheus](prometheus.md)
- [Loki](loki.md)
