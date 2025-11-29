# Storage Setup

Local storage configuration for VMStation.

## Overview

VMStation uses local-path-provisioner for dynamic storage provisioning.

## Local Path Provisioner

### Installation

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Configuration

Storage class configuration:

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

### Node Configuration

Limit provisioning to masternode:

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

## PVC Configuration

### Create PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 5Gi
```

### Verify

```bash
kubectl get pvc -n <namespace>
kubectl get pv
```

## Monitoring PVCs

| PVC | Size | Component |
|-----|------|-----------|
| prometheus-storage-prometheus-0 | 10Gi | Prometheus |
| loki-data-loki-0 | 20Gi | Loki |
| grafana-pvc | 2Gi | Grafana |

## Directory Structure

```
/srv/monitoring_data/
├── pvc-xxx-prometheus/
├── pvc-yyy-loki/
├── pvc-zzz-grafana/
└── ...
```

## Permissions

### Prometheus

```bash
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus
```

### Loki

```bash
sudo chown -R 10001:10001 /srv/monitoring_data/loki
```

### Grafana

```bash
sudo chown -R 472:472 /srv/monitoring_data/grafana
```

## Backup

```bash
# Backup monitoring data
sudo tar -czf backup-$(date +%Y%m%d).tar.gz /srv/monitoring_data/
```

## Troubleshooting

### PVC Pending

```bash
# Check storage class
kubectl get sc

# Check provisioner
kubectl get pods -n local-path-storage

# Check events
kubectl describe pvc <pvc-name> -n <namespace>
```

### Provisioner Not Running

```bash
kubectl rollout restart deployment/local-path-provisioner -n local-path-storage
```

## Related Documentation

- [Storage Architecture](../../architecture/storage-architecture.md)
- [Backup & Restore](../../operations/backup-restore.md)
- [Storage Issues](../../troubleshooting/storage-issues.md)
