# Monitoring Deployment

This guide covers deploying the VMStation monitoring stack.

## Stack Components

| Component | Purpose | Port |
|-----------|---------|------|
| Prometheus | Metrics collection | 30090 |
| Grafana | Visualization | 30300 |
| Loki | Log aggregation | 31100 |
| Promtail | Log shipping | - |
| Node Exporter | System metrics | 9100 |
| Blackbox Exporter | Probes | 9115 |
| kube-state-metrics | K8s metrics | 8080 |
| IPMI Exporter | Hardware metrics | 9290 |

## Quick Deployment

```bash
./deploy.sh monitoring
```

## Manual Deployment

### Apply Manifests

```bash
# Create namespace
kubectl create namespace monitoring

# Apply all monitoring manifests
kubectl apply -f manifests/monitoring/
```

### Individual Components

```bash
# Prometheus
kubectl apply -f manifests/monitoring/prometheus.yaml

# Grafana
kubectl apply -f manifests/monitoring/grafana.yaml

# Loki
kubectl apply -f manifests/monitoring/loki.yaml

# Promtail
kubectl apply -f manifests/monitoring/promtail.yaml

# Node Exporter
kubectl apply -f manifests/monitoring/node-exporter.yaml
```

## Verify Deployment

### Check Pods

```bash
kubectl get pods -n monitoring
```

Expected:
```
NAME                               READY   STATUS    RESTARTS   AGE
prometheus-0                       2/2     Running   0          5m
grafana-xxx                        1/1     Running   0          5m
loki-0                             1/1     Running   0          5m
promtail-xxxxx                     1/1     Running   0          5m
node-exporter-xxxxx                1/1     Running   0          5m
blackbox-exporter-xxx              1/1     Running   0          5m
kube-state-metrics-xxx             1/1     Running   0          5m
```

### Check Services

```bash
kubectl get svc -n monitoring
```

### Check Endpoints

```bash
kubectl get endpoints -n monitoring
```

## Access Services

### Prometheus

```bash
# URL
http://192.168.4.63:30090

# Health check
curl http://192.168.4.63:30090/-/healthy
```

### Grafana

```bash
# URL
http://192.168.4.63:30300

# Health check
curl http://192.168.4.63:30300/api/health
```

Default credentials: admin / admin

### Loki

```bash
# URL
http://192.168.4.63:31100

# Ready check
curl http://192.168.4.63:31100/ready
```

## Validation

### Run Validation Script

```bash
./scripts/validate-monitoring-stack.sh
```

### Check Prometheus Targets

```bash
curl -s http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'
```

### Check Grafana Datasources

```bash
curl -s http://192.168.4.63:30300/api/datasources | jq '.[].name'
```

## Grafana Configuration

### Pre-configured Datasources

- **Prometheus** - `http://prometheus:9090`
- **Loki** - `http://loki:3100`

### Pre-configured Dashboards

- Kubernetes Cluster Overview
- Node Exporter Full
- Loki Logs & Aggregation
- Syslog Infrastructure
- CoreDNS Performance

### Add Custom Dashboard

1. Open Grafana
2. Click + → Import
3. Paste dashboard JSON or ID
4. Select Prometheus datasource
5. Click Import

## Prometheus Configuration

### Scrape Targets

Default targets configured:
- kubernetes-nodes (node-exporter)
- kubernetes-pods (annotated pods)
- kubernetes-service-endpoints
- blackbox-exporter

### Add Custom Target

Edit `manifests/monitoring/prometheus.yaml`:

```yaml
- job_name: 'my-service'
  static_configs:
    - targets: ['my-service:8080']
```

### Reload Configuration

```bash
kubectl delete pod prometheus-0 -n monitoring
```

## Loki Configuration

### Storage

```yaml
storage_config:
  boltdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache
  filesystem:
    directory: /loki/chunks
```

### Retention

```yaml
limits_config:
  retention_period: 744h  # 31 days
```

## Storage Requirements

| Component | Size | Path |
|-----------|------|------|
| Prometheus | 10Gi | /srv/monitoring_data/prometheus |
| Loki | 20Gi | /srv/monitoring_data/loki |
| Grafana | 2Gi | /srv/monitoring_data/grafana |

### Deploy Storage Class

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p \
  '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Troubleshooting

### Pods Not Starting

```bash
# Check events
kubectl describe pod <pod-name> -n monitoring

# Check logs
kubectl logs <pod-name> -n monitoring
```

### Prometheus CrashLoopBackOff

Check SecurityContext:
```yaml
securityContext:
  runAsUser: 65534
  runAsGroup: 65534  # Must be set
  fsGroup: 65534
```

### Loki Not Ready

Check permissions:
```bash
sudo chown -R 10001:10001 /srv/monitoring_data/loki
```

### Grafana Can't Connect

Check DNS resolution:
```bash
kubectl exec -n monitoring deployment/grafana -- nslookup prometheus.monitoring.svc.cluster.local
```

### Run Diagnostics

```bash
./scripts/diagnose-monitoring-stack.sh
```

### Run Remediation

```bash
./scripts/remediate-monitoring-stack.sh
```

## Upgrade Monitoring Stack

### Update Manifests

```bash
git pull
kubectl apply -f manifests/monitoring/
```

### Restart Pods

```bash
kubectl rollout restart deployment/grafana -n monitoring
kubectl rollout restart statefulset/prometheus -n monitoring
kubectl rollout restart statefulset/loki -n monitoring
```

## Backup Monitoring Data

```bash
# Backup Prometheus
sudo tar -czf prometheus-backup.tar.gz /srv/monitoring_data/prometheus

# Backup Loki
sudo tar -czf loki-backup.tar.gz /srv/monitoring_data/loki

# Backup Grafana
sudo tar -czf grafana-backup.tar.gz /srv/monitoring_data/grafana
```

## Related Documentation

- [Monitoring Architecture](../architecture/monitoring-architecture.md)
- [Prometheus Component](../components/monitoring/prometheus.md)
- [Grafana Component](../components/monitoring/grafana.md)
- [Loki Component](../components/monitoring/loki.md)
- [Troubleshooting](../troubleshooting/monitoring-issues.md)
