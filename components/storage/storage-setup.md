# Storage Setup

Storage configuration for VMStation.

## Overview

VMStation uses local-path-provisioner for dynamic storage, with all persistent data on masternode.

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

## Configuration

### Base Path

All persistent data stored in:

```
/srv/monitoring_data/
```

### Node Affinity

Storage provisioned only on masternode:

```yaml
nodePathMap:
  - node: masternode
    paths:
      - /srv/monitoring_data
```

## Current PVCs

| Service | PVC | Capacity | Path |
|---------|-----|----------|------|
| Prometheus | prometheus-storage | 10Gi | /srv/monitoring_data/pvc-xxx/ |
| Loki | loki-data | 20Gi | /srv/monitoring_data/pvc-yyy/ |
| Grafana | grafana-pvc | 2Gi | /srv/monitoring_data/pvc-zzz/ |

## Check Status

```bash
# Storage class
kubectl get sc

# PVCs
kubectl get pvc -n monitoring

# PVs
kubectl get pv
```

## Permissions

### Prometheus

```bash
sudo chown -R 65534:65534 /srv/monitoring_data/*prometheus*/
```

### Loki

```bash
sudo chown -R 10001:10001 /srv/monitoring_data/*loki*/
```

### Grafana

```bash
sudo chown -R 472:472 /srv/monitoring_data/*grafana*/
```

## Retention

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

## Troubleshooting

### PVC Pending

1. Check storage class exists
2. Check provisioner running
3. Check node available

```bash
kubectl get sc
kubectl get pods -n local-path-storage
kubectl get nodes
```

### Permission Denied

Fix ownership per service (see Permissions above).

### Disk Full

```bash
df -h /srv/monitoring_data
du -sh /srv/monitoring_data/pvc-*
```

Options:
- Reduce retention
- Expand disk
- Clean old data

## Related Documentation

- [Storage Architecture](../../architecture/storage-architecture.md)
- [Storage Issues](../../troubleshooting/storage-issues.md)
- [Backup & Restore](../../operations/backup-restore.md)
