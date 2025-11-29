# Exporters

Prometheus exporters collect metrics from various sources.

## Overview

| Exporter | Purpose | Port |
|----------|---------|------|
| Node Exporter | System metrics | 9100 |
| Blackbox Exporter | Probes | 9115 |
| IPMI Exporter | Hardware | 9290 |
| kube-state-metrics | K8s objects | 8080 |

## Node Exporter

Collects host-level system metrics.

### Deployment

Runs as DaemonSet on all nodes.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  template:
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
```

### Key Metrics

```promql
# CPU usage
rate(node_cpu_seconds_total{mode!="idle"}[5m])

# Memory available
node_memory_MemAvailable_bytes

# Disk usage
node_filesystem_avail_bytes

# Network traffic
rate(node_network_receive_bytes_total[5m])
```

### Access

```bash
curl http://192.168.4.63:9100/metrics
```

## Blackbox Exporter

Probes endpoints for availability.

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blackbox-exporter
  namespace: monitoring
```

### Probe Types

- HTTP
- HTTPS
- TCP
- DNS
- ICMP

### Configuration

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: [200]
      
  tcp_connect:
    prober: tcp
    timeout: 5s
    
  dns_check:
    prober: dns
    timeout: 5s
    dns:
      query_name: kubernetes.default.svc.cluster.local
```

### Prometheus Scrape Config

```yaml
- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
        - http://192.168.4.63:30300
        - http://192.168.4.63:30090
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: blackbox-exporter:9115
```

## IPMI Exporter

Collects hardware metrics via IPMI.

### Requirements

- IPMI/BMC access
- IPMI credentials
- freeipmi or ipmitool

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ipmi-exporter
  namespace: monitoring
spec:
  template:
    spec:
      containers:
      - name: ipmi-exporter
        image: prometheuscommunity/ipmi-exporter:latest
        ports:
        - containerPort: 9290
```

### Metrics

- Temperature sensors
- Fan speeds
- Power consumption
- Voltage readings

### Status

Currently optional - requires IPMI credentials.

## kube-state-metrics

Exposes Kubernetes object state.

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: monitoring
```

### Key Metrics

```promql
# Pod status
kube_pod_status_phase{phase="Running"}

# Deployment replicas
kube_deployment_status_replicas

# Node conditions
kube_node_status_condition

# PVC status
kube_persistentvolumeclaim_status_phase
```

## Promtail

Ships logs to Loki.

### Deployment

Runs as DaemonSet.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: promtail
  namespace: monitoring
```

### Configuration

```yaml
scrape_configs:
  - job_name: kubernetes-pods
    kubernetes_sd_configs:
      - role: pod
    pipeline_stages:
      - docker: {}
```

### Log Sources

- `/var/log/containers/*.log`
- `/var/log/pods/*/*.log`

## Adding Custom Exporters

### Create Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-exporter
  namespace: monitoring
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9999"
```

### Add Scrape Config

```yaml
- job_name: 'my-exporter'
  static_configs:
    - targets: ['my-exporter:9999']
```

## Troubleshooting

### Exporter Down

```bash
# Check pod
kubectl get pods -n monitoring -l app=<exporter>

# Check service
kubectl get endpoints -n monitoring <exporter>
```

### No Metrics

```bash
# Test directly
curl http://<node>:9100/metrics

# Check Prometheus targets
curl http://192.168.4.63:30090/api/v1/targets | jq
```

## Related Documentation

- [Prometheus](prometheus.md)
- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
