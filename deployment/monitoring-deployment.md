# Monitoring Deployment

Deploy the complete monitoring and observability stack.

## Overview

The monitoring stack includes:
- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **Loki** - Log aggregation
- **Promtail** - Log shipping
- **Node Exporter** - System metrics
- **Blackbox Exporter** - Probes

## Quick Deployment

```bash
./deploy.sh monitoring
```

## Components

### Prometheus

| Property | Value |
|----------|-------|
| Port | 30090 (NodePort) |
| Storage | 10Gi PVC |
| Retention | 15 days |

### Grafana

| Property | Value |
|----------|-------|
| Port | 30300 (NodePort) |
| Storage | 2Gi PVC |
| Auth | Anonymous enabled (Viewer) |

### Loki

| Property | Value |
|----------|-------|
| Port | 31100 (NodePort) |
| Storage | 20Gi PVC |
| Retention | 31 days |

### Promtail

| Property | Value |
|----------|-------|
| Deployment | DaemonSet |
| Sources | /var/log/containers/*.log |

### Node Exporter

| Property | Value |
|----------|-------|
| Port | 9100 |
| Deployment | DaemonSet |

### Blackbox Exporter

| Property | Value |
|----------|-------|
| Port | 9115 |
| Probes | HTTP, TCP, DNS |

## Manual Deployment

### Deploy Prometheus

```bash
kubectl apply -f manifests/monitoring/prometheus.yaml
```

### Deploy Grafana

```bash
kubectl apply -f manifests/monitoring/grafana.yaml
```

### Deploy Loki

```bash
kubectl apply -f manifests/monitoring/loki.yaml
```

### Deploy Promtail

```bash
kubectl apply -f manifests/monitoring/promtail.yaml
```

### Deploy Exporters

```bash
kubectl apply -f manifests/monitoring/node-exporter.yaml
kubectl apply -f manifests/monitoring/blackbox-exporter.yaml
kubectl apply -f manifests/monitoring/kube-state-metrics.yaml
```

## Storage Requirements

Before deploying, ensure storage class is configured:

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Verification

### Check Pods

```bash
kubectl get pods -n monitoring
```

Expected output:
```
NAME                                  READY   STATUS    RESTARTS   AGE
prometheus-0                          2/2     Running   0          5m
loki-0                                1/1     Running   0          5m
grafana-5f879c7654-dnmhs              1/1     Running   0          5m
node-exporter-xxxxx                   1/1     Running   0          5m
promtail-xxxxx                        1/1     Running   0          5m
```

### Check Services

```bash
kubectl get svc -n monitoring
```

### Test Endpoints

```bash
curl http://192.168.4.63:30090/-/healthy  # Prometheus
curl http://192.168.4.63:31100/ready      # Loki
curl http://192.168.4.63:30300/api/health # Grafana
```

### Run Validation Script

```bash
./scripts/validate-monitoring-stack.sh
```

## Grafana Configuration

### Access Grafana

URL: http://192.168.4.63:30300

Default credentials: admin/admin

### Pre-configured Dashboards

- Kubernetes Cluster Overview
- Node Exporter Full
- Loki Logs
- Syslog Infrastructure
- CoreDNS Performance

### Datasources

Datasources are pre-configured:
- Prometheus: http://prometheus:9090
- Loki: http://loki:3100

## Troubleshooting

### Pod CrashLoopBackOff

**Prometheus:**
```bash
# Check if runAsGroup is set
kubectl get statefulset prometheus -n monitoring -o yaml | grep -A5 securityContext
```

Fix: Add `runAsGroup: 65534` to securityContext

**Loki:**
```bash
# Check frontend_worker config
kubectl get configmap loki-config -n monitoring -o yaml
```

Fix: Disable `frontend_worker` for single-instance

### Empty Endpoints

```bash
# Check endpoints
kubectl get endpoints -n monitoring

# Endpoints populate when pods are Ready
kubectl get pods -n monitoring
```

### Storage Issues

```bash
# Check PVC status
kubectl get pvc -n monitoring

# Check permissions
ls -la /srv/monitoring_data/pvc-*/
```

See [Monitoring Issues](../troubleshooting/monitoring-issues.md) for detailed troubleshooting.

## Related Documentation

- [Monitoring Architecture](../architecture/monitoring-architecture.md)
- [Monitoring Issues](../troubleshooting/monitoring-issues.md)
- [Prometheus](../components/monitoring/prometheus.md)
- [Grafana](../components/monitoring/grafana.md)
- [Loki](../components/monitoring/loki.md)
