# Storage Issues

This guide covers storage-related troubleshooting.

## PVC Issues

### PVC Stuck Pending

**Symptoms:**
```bash
kubectl get pvc -n monitoring
NAME                              STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS
prometheus-storage-prometheus-0   Pending                                      
```

**Diagnosis:**
```bash
kubectl describe pvc prometheus-storage-prometheus-0 -n monitoring
```

**Common Causes:**
1. No storage class
2. Storage class not default
3. No available storage

**Solution - Deploy Storage Class:**
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Volume Mount Failed

**Symptoms:**
```
Unable to attach or mount volumes: timed out waiting for the condition
```

**Diagnosis:**
```bash
kubectl describe pod <pod> -n <namespace>
```

**Solutions:**
```bash
# Check PV status
kubectl get pv

# Check node has storage
ssh <node> "df -h /srv/monitoring_data"

# Restart pod
kubectl delete pod <pod> -n <namespace>
```

## Permission Issues

### Permission Denied

**Symptoms:**
```
permission denied: /prometheus/lock
mkdir /loki/chunks: permission denied
```

**Solutions by Component:**

**Prometheus:**
```bash
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus
```

**Loki:**
```bash
sudo chown -R 10001:10001 /srv/monitoring_data/loki
```

**Grafana:**
```bash
sudo chown -R 472:472 /srv/monitoring_data/grafana
```

### SecurityContext Issues

**Symptoms:**
- Pod can't write to volume
- Permission denied despite correct ownership

**Solution:**

Add runAsGroup to SecurityContext:
```yaml
securityContext:
  runAsUser: 65534
  runAsGroup: 65534  # Add this
  fsGroup: 65534
```

## Disk Space Issues

### Disk Full

**Symptoms:**
```
no space left on device
```

**Diagnosis:**
```bash
df -h /srv/monitoring_data
du -sh /srv/monitoring_data/*
```

**Solutions:**

**Cleanup old data:**
```bash
# Prometheus handles retention automatically
# For manual cleanup:
sudo rm -rf /srv/monitoring_data/prometheus/wal/*

# Loki cleanup
sudo rm -rf /srv/monitoring_data/loki/chunks/old/*
```

**Expand storage:**
1. Add disk to node
2. Expand partition
3. Resize filesystem

### Retention Configuration

**Prometheus:**
```yaml
args:
  - '--storage.tsdb.retention.time=15d'
  - '--storage.tsdb.retention.size=8GB'
```

**Loki:**
```yaml
limits_config:
  retention_period: 744h  # 31 days
```

## Local Path Provisioner Issues

### Provisioner Not Running

**Diagnosis:**
```bash
kubectl get pods -n local-path-storage
kubectl logs -n local-path-storage -l app=local-path-provisioner
```

**Solution:**
```bash
kubectl rollout restart deployment/local-path-provisioner -n local-path-storage
```

### Wrong Node

**Symptoms:**
- PV created on wrong node
- Pod can't mount volume

**Solution:**

Configure node affinity in provisioner:
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

## TSDB Corruption

### Prometheus TSDB Issues

**Symptoms:**
```
out of sequence m-mapped chunk for series ref
```

**Solution:**

Prometheus auto-repairs. If persistent:
```bash
# Backup
sudo tar -czf prometheus-backup.tar.gz /srv/monitoring_data/prometheus

# Clear and restart
kubectl scale statefulset prometheus -n monitoring --replicas=0
sudo rm -rf /srv/monitoring_data/prometheus/*
kubectl scale statefulset prometheus -n monitoring --replicas=1
```

### Loki Index Corruption

**Symptoms:**
- Query failures
- Missing logs

**Solution:**
```bash
# Clear cache
sudo rm -rf /srv/monitoring_data/loki/cache/*
kubectl delete pod loki-0 -n monitoring
```

## Backup Verification

### Check Backup Integrity

```bash
# Test tar extraction
tar -tzf /backup/monitoring/prometheus-20251014.tar.gz | head

# Verify file count
tar -tzf /backup/monitoring/prometheus-20251014.tar.gz | wc -l
```

### Restore Test

```bash
# Extract to temp directory
mkdir /tmp/restore-test
tar -xzf /backup/monitoring/prometheus-20251014.tar.gz -C /tmp/restore-test
ls -la /tmp/restore-test/srv/monitoring_data/prometheus/
```

## Storage Monitoring

### Check Usage

```bash
# Overall
df -h /srv/monitoring_data

# Per PVC directory
du -sh /srv/monitoring_data/pvc-*
```

### Prometheus Metrics

```promql
# Disk available
node_filesystem_avail_bytes{mountpoint="/srv/monitoring_data"}

# Prometheus TSDB size
prometheus_tsdb_storage_blocks_bytes
```

## Related Documentation

- [Storage Architecture](../architecture/storage-architecture.md)
- [Backup & Restore](../operations/backup-restore.md)
- [Common Issues](common-issues.md)
