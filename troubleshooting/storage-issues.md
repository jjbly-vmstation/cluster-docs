# Storage Issues

Troubleshooting storage and volume problems.

## PVC Pending

### Symptoms

```bash
kubectl get pvc -n monitoring
# STATUS: Pending
```

### Diagnosis

```bash
kubectl describe pvc <pvc-name> -n <namespace>
```

### Common Causes

#### No Storage Class

```bash
kubectl get sc
# Empty output = no storage class
```

**Fix:**
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

#### Provisioner Not Running

```bash
kubectl get pods -n local-path-storage
```

**Fix:**
```bash
kubectl rollout restart deployment local-path-provisioner -n local-path-storage
```

#### Node Not Available

Storage is only on masternode. If masternode is down, PVCs can't bind.

```bash
kubectl get nodes
```

#### Volume Binding Mode

```bash
kubectl get sc local-path -o yaml | grep volumeBindingMode
# WaitForFirstConsumer = binds when pod scheduled
```

## Permission Denied

### Symptoms

```
Error: opening storage failed: permission denied
Error: unable to create file: permission denied
```

### Fix Permissions

```bash
# Prometheus
sudo chown -R 65534:65534 /srv/monitoring_data/*prometheus*/

# Loki
sudo chown -R 10001:10001 /srv/monitoring_data/*loki*/

# Grafana
sudo chown -R 472:472 /srv/monitoring_data/*grafana*/
```

### SecurityContext

Ensure pod has correct SecurityContext:

```yaml
securityContext:
  runAsUser: 65534
  runAsGroup: 65534  # Important!
  fsGroup: 65534
```

## Disk Full

### Symptoms

- Pods failing with disk errors
- Metrics/logs not persisting

### Check Usage

```bash
df -h /srv/monitoring_data
du -sh /srv/monitoring_data/pvc-*
```

### Fix

1. **Increase retention settings**
   ```yaml
   # Prometheus
   args:
     - '--storage.tsdb.retention.size=4GB'
   
   # Loki
   limits_config:
     retention_period: 168h  # 7 days
   ```

2. **Clean old data**
   ```bash
   # Dangerous - backup first!
   rm -rf /srv/monitoring_data/pvc-*/old-data
   ```

3. **Expand disk** (if possible)

## Volume Not Mounting

### Symptoms

Pod stuck in ContainerCreating

### Diagnosis

```bash
kubectl describe pod <pod> -n <namespace>
# Look for mount errors
```

### Common Causes

#### Path doesn't exist

```bash
ls -la /srv/monitoring_data/
```

**Fix:**
```bash
mkdir -p /srv/monitoring_data
```

#### Wrong permissions on host path

```bash
ls -la /srv/monitoring_data/pvc-*/
```

#### Previous data conflict

```bash
# Remove conflicting data
rm -rf /srv/monitoring_data/pvc-*/lock*
```

## Storage Class Issues

### Check Storage Class

```bash
kubectl get sc
kubectl describe sc local-path
```

### Make Default

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Provisioner Logs

```bash
kubectl logs -n local-path-storage -l app=local-path-provisioner
```

## Data Recovery

### Backup Before Changes

```bash
tar -czf /backup/pvc-backup.tar.gz /srv/monitoring_data/
```

### Restore from Backup

See [Backup & Restore](../operations/backup-restore.md)

## Storage Capacity

### Current Allocation

| Service | PVC | Capacity |
|---------|-----|----------|
| Prometheus | prometheus-storage | 10Gi |
| Loki | loki-data | 20Gi |
| Grafana | grafana-pvc | 2Gi |

### Expand PVC

```bash
# If storage class supports expansion
kubectl patch pvc <pvc-name> -n <namespace> -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'
```

## Related Documentation

- [Storage Architecture](../architecture/storage-architecture.md)
- [Backup & Restore](../operations/backup-restore.md)
- [Monitoring Issues](monitoring-issues.md)
