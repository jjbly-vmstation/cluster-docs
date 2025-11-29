# Syslog

Centralized log collection.

## Overview

Syslog collection is provided by Promtail shipping logs to Loki.

## Current Setup

### Promtail

Promtail (deployed as DaemonSet) collects:
- Container logs: `/var/log/containers/*.log`
- System logs: `/var/log/syslog`

### Loki

Loki stores and indexes all logs.

### Grafana

Query logs via Grafana Explore.

## Querying Syslog

### Basic Query

```logql
{job="syslog"}
```

### Filter by Severity

```logql
{job="syslog"} | json | severity="error"
```

### Handle Parse Errors

```logql
{job="syslog"} | json | __error__=""
```

## Alternative: Dedicated Syslog Server

For traditional syslog reception:

### Deploy rsyslog

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: syslog-server
  namespace: infrastructure
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: rsyslog
        image: rsyslog/syslog_appliance_alpine:latest
        ports:
        - containerPort: 514
          protocol: UDP
        - containerPort: 514
          protocol: TCP
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: syslog
  namespace: infrastructure
spec:
  type: NodePort
  ports:
  - port: 514
    targetPort: 514
    protocol: UDP
    nodePort: 30514
```

### Client Configuration

On remote hosts:

```bash
# /etc/rsyslog.d/50-remote.conf
*.* @192.168.4.63:30514
```

## Troubleshooting

### No Syslog Logs

1. Check Promtail configuration
2. Verify syslog file exists
3. Check Promtail logs

```bash
kubectl logs -n monitoring -l app=promtail | grep syslog
```

### Parse Errors

JSON parse errors usually mean incomplete messages.

Use error filtering in queries:
```logql
{job="syslog"} | json | __error__=""
```

## Dashboard

Syslog Infrastructure dashboard available in Grafana.

## Related Documentation

- [Loki](../monitoring/loki.md)
- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
