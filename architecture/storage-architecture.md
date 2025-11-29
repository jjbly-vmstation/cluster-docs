# Storage Architecture

VMStation uses local storage on the control plane node for persistent data.

## Storage Design

### Design Principles

1. **Centralized Storage** - All persistent data on masternode
2. **Stateless Workers** - Worker nodes are ephemeral
3. **Local Path Provisioner** - Dynamic PV provisioning
4. **Retain Policy** - Data preserved on PVC deletion

### Storage Topology

```
┌─────────────────────────────────────────────────────────────────┐
│                    Storage Architecture                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ masternode (192.168.4.63)                                   │ │
│  │                                                             │ │
│  │  /srv/monitoring_data/                                      │ │
│  │    ├── prometheus/  (metrics data)                          │ │
│  │    ├── loki/        (log data)                              │ │
│  │    ├── grafana/     (dashboards, users)                     │ │
│  │    └── pvc-*/       (dynamically provisioned)               │ │
│  │                                                             │ │
│  │  Storage Class: local-path (default)                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ storagenodet3500 (192.168.4.61)                            │ │
│  │                                                             │ │
│  │  /media/                                                    │ │
│  │    ├── movies/      (Jellyfin media)                        │ │
│  │    ├── tv/          (TV shows)                              │ │
│  │    └── music/       (Audio files)                           │ │
│  │                                                             │ │
│  │  No Kubernetes PVs - direct hostPath                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Local Path Provisioner

### Installation

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p \
  '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Configuration

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

### How It Works

1. PVC created in Kubernetes
2. Pod scheduled on node
3. Local-path-provisioner creates directory
4. PV bound to PVC
5. Directory mounted in pod

## Persistent Volume Claims

### Monitoring PVCs

| Service | PVC Name | Size | Status |
|---------|----------|------|--------|
| Prometheus | prometheus-storage-prometheus-0 | 10Gi | Bound |
| Loki | loki-data-loki-0 | 20Gi | Bound |
| Grafana | grafana-pvc | 2Gi | Bound |

### Check PVC Status

```bash
kubectl get pvc -n monitoring

NAME                               STATUS   VOLUME         CAPACITY   ACCESS MODES
prometheus-storage-prometheus-0    Bound    pvc-xxx        10Gi       RWO
loki-data-loki-0                   Bound    pvc-yyy        20Gi       RWO
grafana-pvc                        Bound    pvc-zzz        2Gi        RWO
```

### Check PV Status

```bash
kubectl get pv

NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
pvc-xxx     10Gi       RWO            Retain           Bound    monitoring/prometheus-storage-prometheus-0
pvc-yyy     20Gi       RWO            Retain           Bound    monitoring/loki-data-loki-0
pvc-zzz     2Gi        RWO            Retain           Bound    monitoring/grafana-pvc
```

## Data Retention

### Prometheus

```yaml
# In prometheus.yaml StatefulSet
args:
  - '--storage.tsdb.retention.time=15d'
  - '--storage.tsdb.retention.size=8GB'
```

### Loki

```yaml
# In loki ConfigMap
limits_config:
  retention_period: 744h  # 31 days
```

### Grafana

- No automatic cleanup
- Manual dashboard management
- User data persisted

## Backup Procedures

### Create Backup

```bash
# Create backup directory
sudo mkdir -p /backup/monitoring

# Backup Prometheus
sudo tar -czf /backup/monitoring/prometheus-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/prometheus/

# Backup Loki
sudo tar -czf /backup/monitoring/loki-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/loki/

# Backup Grafana
sudo tar -czf /backup/monitoring/grafana-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/grafana/
```

### Restore from Backup

```bash
# Stop services
kubectl scale statefulset prometheus -n monitoring --replicas=0
kubectl scale statefulset loki -n monitoring --replicas=0
kubectl scale deployment grafana -n monitoring --replicas=0

# Restore data
sudo tar -xzf /backup/monitoring/prometheus-20251014.tar.gz -C /

# Restart services
kubectl scale statefulset prometheus -n monitoring --replicas=1
kubectl scale statefulset loki -n monitoring --replicas=1
kubectl scale deployment grafana -n monitoring --replicas=1
```

## Media Storage (Jellyfin)

### Directory Structure

```
/media/
├── movies/
├── tv/
├── music/
└── photos/
```

### Permissions

```bash
# Set ownership
sudo chown -R 1000:1000 /media

# Set permissions
sudo chmod -R 755 /media
```

### Volume Mount in Jellyfin

```yaml
volumeMounts:
  - name: media
    mountPath: /media
volumes:
  - name: media
    hostPath:
      path: /media
      type: Directory
```

## Storage Monitoring

### Check Disk Usage

```bash
# On masternode
df -h /srv/monitoring_data

# Per PVC
du -sh /srv/monitoring_data/pvc-*
```

### Prometheus Metrics

```promql
# Disk usage
node_filesystem_avail_bytes{mountpoint="/srv/monitoring_data"}

# TSDB size
prometheus_tsdb_storage_blocks_bytes
```

### Alerts

Set up alerts for:
- Disk usage > 80%
- PVC usage approaching limit
- TSDB compaction issues

## Troubleshooting

### PVC Stuck Pending

```bash
# Check storage class
kubectl get sc

# Check provisioner
kubectl get pods -n local-path-storage

# Check events
kubectl describe pvc <pvc-name> -n <namespace>
```

### Permission Issues

```bash
# Check directory ownership
ls -la /srv/monitoring_data/

# Fix Loki permissions
sudo chown -R 10001:10001 /srv/monitoring_data/loki

# Fix Prometheus permissions
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus
```

### Data Corruption

```bash
# Check Prometheus TSDB
kubectl logs prometheus-0 -n monitoring -c prometheus | grep -i corrupt

# Prometheus will auto-repair minor corruption
# For major issues, restore from backup
```

## Related Documentation

- [Architecture Overview](overview.md)
- [Monitoring Architecture](monitoring-architecture.md)
- [Storage Setup](../components/storage/storage-setup.md)
