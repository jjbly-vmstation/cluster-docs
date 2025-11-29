# Exporters

Metric exporters for Prometheus.

## Node Exporter

System metrics exporter.

### Deployment

Runs as DaemonSet on all nodes.

| Property | Value |
|----------|-------|
| Port | 9100 |
| Path | /metrics |

### Metrics

```promql
# CPU usage
node_cpu_seconds_total

# Memory
node_memory_MemTotal_bytes
node_memory_MemAvailable_bytes

# Disk
node_filesystem_size_bytes
node_filesystem_avail_bytes

# Network
node_network_receive_bytes_total
node_network_transmit_bytes_total
```

### Test

```bash
curl http://192.168.4.63:9100/metrics | head -20
```

## Blackbox Exporter

Probe endpoints for availability.

### Deployment

Single replica on masternode.

| Property | Value |
|----------|-------|
| Port | 9115 |
| Probes | HTTP, TCP, DNS |

### Configuration

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      method: GET
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      valid_status_codes: [200]
  
  dns_check:
    prober: dns
    timeout: 5s
    dns:
      query_name: kubernetes.default.svc.cluster.local
      query_type: A
```

### Usage

```promql
probe_success{job="blackbox"}
probe_http_duration_seconds
```

### Test

```bash
curl "http://192.168.4.63:9115/probe?target=http://google.com&module=http_2xx"
```

## Kube-state-metrics

Kubernetes object state metrics.

### Deployment

Single replica on masternode.

| Property | Value |
|----------|-------|
| Port | 8080 |

### Metrics

```promql
# Pod status
kube_pod_status_phase

# Deployment status
kube_deployment_status_replicas
kube_deployment_status_replicas_available

# Node status
kube_node_status_condition

# PVC status
kube_persistentvolumeclaim_status_phase
```

### Test

```bash
curl http://<kube-state-metrics-ip>:8080/metrics | grep kube_pod
```

## IPMI Exporter

Hardware metrics via IPMI.

### Status

**Not deployed** - Requires IPMI credentials.

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ipmi-exporter
spec:
  template:
    spec:
      containers:
      - name: ipmi-exporter
        image: prometheuscommunity/ipmi-exporter:latest
        ports:
        - containerPort: 9290
```

### Configuration

Requires IPMI credentials in secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ipmi-credentials
data:
  username: <base64>
  password: <base64>
```

### Metrics

```promql
ipmi_temperature_celsius
ipmi_fan_speed_rpm
ipmi_power_watts
ipmi_voltage_volts
```

## Adding New Exporters

### 1. Deploy Exporter

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-exporter
  namespace: monitoring
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: exporter
        image: my-exporter:latest
        ports:
        - containerPort: 9xxx
```

### 2. Create Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-exporter
  namespace: monitoring
spec:
  ports:
  - port: 9xxx
  selector:
    app: my-exporter
```

### 3. Add Scrape Config

Update Prometheus ConfigMap:

```yaml
scrape_configs:
  - job_name: my-exporter
    static_configs:
      - targets:
        - my-exporter:9xxx
```

### 4. Reload Prometheus

```bash
curl -X POST http://192.168.4.63:30090/-/reload
```

## Troubleshooting

### Exporter Not in Targets

1. Check pod running
2. Check service created
3. Check scrape config
4. Reload Prometheus

### Metrics Missing

1. Verify exporter accessible
2. Check /metrics endpoint
3. Review Prometheus logs

## Related Documentation

- [Prometheus](prometheus.md)
- [Monitoring Architecture](../../architecture/monitoring-architecture.md)
- [Monitoring Deployment](../../deployment/monitoring-deployment.md)
