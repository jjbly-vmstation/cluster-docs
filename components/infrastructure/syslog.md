# Syslog

Centralized log collection for system-level logs.

## Overview

Syslog collection in VMStation is handled by:
1. **Promtail** (current) - Ships logs to Loki
2. **Dedicated syslog server** (optional)

## Promtail Syslog Collection

Promtail already runs as a DaemonSet and can collect syslog.

### Configuration

Add to promtail scrape config:

```yaml
scrape_configs:
  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          __path__: /var/log/syslog
```

### Query Syslog in Grafana

```logql
{job="syslog"}
```

## Dedicated Syslog Server (Optional)

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: syslog-server
  namespace: infrastructure
spec:
  replicas: 1
  selector:
    matchLabels:
      app: syslog
  template:
    metadata:
      labels:
        app: syslog
    spec:
      containers:
      - name: rsyslog
        image: rsyslog/syslog_appliance_alpine:latest
        ports:
        - containerPort: 514
          protocol: UDP
        - containerPort: 514
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: syslog
  namespace: infrastructure
spec:
  type: NodePort
  selector:
    app: syslog
  ports:
  - name: syslog-udp
    port: 514
    targetPort: 514
    protocol: UDP
    nodePort: 30514
```

### Configure Nodes

Forward logs to syslog server:

```bash
# /etc/rsyslog.d/50-forward.conf
*.* @192.168.4.63:30514
```

## Syslog Dashboard

A pre-configured dashboard shows:
- Message rate
- Severity distribution
- Facility breakdown
- Recent events

Access in Grafana: Syslog Infrastructure Monitoring

## Troubleshooting

### Logs Not Appearing

```bash
# Check promtail logs
kubectl logs -n monitoring -l app=promtail

# Test syslog connectivity
nc -u 192.168.4.63 30514 <<< "test message"
```

### Parse Errors

Use error filtering:
```logql
{job="syslog"} | json | __error__=""
```

## Related Documentation

- [Loki](../monitoring/loki.md)
- [Infrastructure Services](../../deployment/infrastructure-services.md)
