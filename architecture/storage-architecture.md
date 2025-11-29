# Storage Architecture

Storage configuration and design for VMStation.

## Storage Overview

VMStation uses local-path-provisioner for dynamic storage provisioning, with all persistent data stored on masternode.

```
┌──────────────────────────────────────────────────────────┐
│                    Storage Architecture                   │
├──────────────────────────────────────────────────────────┤
│  masternode (192.168.4.63)                               │
│  ┌────────────────────────────────────────────────────┐  │
│  │  /srv/monitoring_data/                              │  │
│  │  ├── pvc-xxx.../  (Prometheus - 10Gi)              │  │
│  │  ├── pvc-yyy.../  (Loki - 20Gi)                    │  │
│  │  └── pvc-zzz.../  (Grafana - 2Gi)                  │  │
│  └────────────────────────────────────────────────────┘  │
├──────────────────────────────────────────────────────────┤
│  Worker Nodes                                            │
│  - Stateless (no persistent storage)                     │
│  - Run exporters only (node-exporter, promtail)          │
└──────────────────────────────────────────────────────────┘
```

## Storage Class

### Local Path Provisioner

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
```

### Installation

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Configuration

Node selector limits provisioning to masternode:

```yaml
data:
  config.json: |
    {
      "nodePathMap":[
        {
          "node":"masternode",
          "paths":["/srv/monitoring_data"]
        }
      ]
    }
```

## Persistent Volume Claims

### Current PVCs

| Service | PVC Name | Capacity | Status |
|---------|----------|----------|--------|
| Prometheus | prometheus-storage-prometheus-0 | 10Gi | Bound |
| Loki | loki-data-loki-0 | 20Gi | Bound |
| Grafana | grafana-pvc | 2Gi | Bound |

### Check PVC Status

```bash
kubectl get pvc -n monitoring
kubectl get pv
```

## Data Retention

### Prometheus

```yaml
args:
  - '--storage.tsdb.retention.time=15d'
  - '--storage.tsdb.retention.size=8GB'
```

### Loki

```yaml
limits_config:
  retention_period: 744h  # 31 days
```

## Storage Paths

### Directory Structure

```
/srv/monitoring_data/
├── pvc-1f0d9b26.../    # Prometheus data
├── pvc-88694ca1.../    # Loki data
└── pvc-67d9eff0.../    # Grafana data
```

### Permissions

```bash
# Prometheus
sudo chown -R 65534:65534 /srv/monitoring_data/pvc-*prometheus*

# Loki
sudo chown -R 10001:10001 /srv/monitoring_data/pvc-*loki*

# Grafana
sudo chown -R 472:472 /srv/monitoring_data/pvc-*grafana*
```

## Backup Strategy

### Manual Backup

```bash
# Create backup directory
sudo mkdir -p /backup/monitoring

# Backup Prometheus
sudo tar -czf /backup/monitoring/prometheus-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/*prometheus*/

# Backup Loki
sudo tar -czf /backup/monitoring/loki-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/*loki*/

# Backup Grafana
sudo tar -czf /backup/monitoring/grafana-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/*grafana*/
```

### Restore

```bash
# Stop services
kubectl scale statefulset prometheus -n monitoring --replicas=0
kubectl scale statefulset loki -n monitoring --replicas=0
kubectl scale deployment grafana -n monitoring --replicas=0

# Restore data
sudo tar -xzf /backup/monitoring/prometheus-YYYYMMDD.tar.gz -C /

# Start services
kubectl scale statefulset prometheus -n monitoring --replicas=1
kubectl scale statefulset loki -n monitoring --replicas=1
kubectl scale deployment grafana -n monitoring --replicas=1
```

## Storage Monitoring

### Check Usage

```bash
df -h /srv/monitoring_data
du -sh /srv/monitoring_data/pvc-*
```

### Prometheus Metrics

```promql
# Disk usage
node_filesystem_avail_bytes{mountpoint="/srv/monitoring_data"}

# TSDB size
prometheus_tsdb_storage_blocks_bytes
```

## Troubleshooting

### PVC Pending

```bash
# Check storage class
kubectl get sc

# Check provisioner
kubectl get pods -n local-path-storage

# Check events
kubectl describe pvc <pvc-name> -n monitoring
```

### Permission Denied

```bash
# Check ownership
ls -la /srv/monitoring_data/pvc-*/

# Fix Prometheus
sudo chown -R 65534:65534 /srv/monitoring_data/pvc-*prometheus*
```

## Related Documentation

- [Architecture Overview](overview.md)
- [Storage Issues](../troubleshooting/storage-issues.md)
- [Backup & Restore](../operations/backup-restore.md)
