# Backup & Restore

Data protection procedures for VMStation.

## Backup Strategy

### What to Backup

| Component | Location | Priority |
|-----------|----------|----------|
| Prometheus | /srv/monitoring_data/pvc-*prometheus* | High |
| Loki | /srv/monitoring_data/pvc-*loki* | High |
| Grafana | /srv/monitoring_data/pvc-*grafana* | High |
| etcd | /var/lib/etcd | Critical |
| Manifests | Repository | High |

### Backup Schedule

| Frequency | Components |
|-----------|------------|
| Daily | Grafana dashboards, configurations |
| Weekly | Prometheus, Loki data |
| Monthly | Full system backup |

## Monitoring Data Backup

### Manual Backup

```bash
# Create backup directory
sudo mkdir -p /backup/monitoring/$(date +%Y%m%d)

# Stop services for consistency (optional)
kubectl scale statefulset prometheus -n monitoring --replicas=0
kubectl scale statefulset loki -n monitoring --replicas=0
kubectl scale deployment grafana -n monitoring --replicas=0

# Backup Prometheus
sudo tar -czf /backup/monitoring/$(date +%Y%m%d)/prometheus.tar.gz \
  /srv/monitoring_data/*prometheus*/

# Backup Loki
sudo tar -czf /backup/monitoring/$(date +%Y%m%d)/loki.tar.gz \
  /srv/monitoring_data/*loki*/

# Backup Grafana
sudo tar -czf /backup/monitoring/$(date +%Y%m%d)/grafana.tar.gz \
  /srv/monitoring_data/*grafana*/

# Restart services
kubectl scale statefulset prometheus -n monitoring --replicas=1
kubectl scale statefulset loki -n monitoring --replicas=1
kubectl scale deployment grafana -n monitoring --replicas=1
```

### Automated Backup Script

```bash
#!/bin/bash
BACKUP_DIR=/backup/monitoring/$(date +%Y%m%d)
mkdir -p $BACKUP_DIR

# Backup without stopping services (less consistent but no downtime)
tar -czf $BACKUP_DIR/prometheus.tar.gz /srv/monitoring_data/*prometheus*/ 2>/dev/null
tar -czf $BACKUP_DIR/loki.tar.gz /srv/monitoring_data/*loki*/ 2>/dev/null
tar -czf $BACKUP_DIR/grafana.tar.gz /srv/monitoring_data/*grafana*/ 2>/dev/null

# Cleanup old backups (keep 7 days)
find /backup/monitoring -type d -mtime +7 -exec rm -rf {} \; 2>/dev/null

echo "Backup completed: $BACKUP_DIR"
```

Add to cron:
```bash
0 2 * * * /root/scripts/backup-monitoring.sh >> /var/log/backup.log 2>&1
```

## Grafana Dashboard Export

### Export via UI

1. Open Grafana: http://192.168.4.63:30300
2. Go to Dashboard → Settings → JSON Model
3. Copy JSON and save to file

### Export via API

```bash
# Get dashboard list
curl -s http://192.168.4.63:30300/api/search | jq '.[].uid'

# Export dashboard
curl -s http://192.168.4.63:30300/api/dashboards/uid/<uid> > dashboard.json
```

## etcd Backup

### Manual Snapshot

```bash
# On masternode
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd/snapshot-$(date +%Y%m%d).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### Verify Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd/snapshot.db
```

## Restore Procedures

### Restore Prometheus

```bash
# Stop Prometheus
kubectl scale statefulset prometheus -n monitoring --replicas=0

# Restore data
tar -xzf /backup/monitoring/YYYYMMDD/prometheus.tar.gz -C /

# Fix permissions
sudo chown -R 65534:65534 /srv/monitoring_data/*prometheus*/

# Start Prometheus
kubectl scale statefulset prometheus -n monitoring --replicas=1
```

### Restore Loki

```bash
# Stop Loki
kubectl scale statefulset loki -n monitoring --replicas=0

# Restore data
tar -xzf /backup/monitoring/YYYYMMDD/loki.tar.gz -C /

# Fix permissions
sudo chown -R 10001:10001 /srv/monitoring_data/*loki*/

# Start Loki
kubectl scale statefulset loki -n monitoring --replicas=1
```

### Restore Grafana

```bash
# Stop Grafana
kubectl scale deployment grafana -n monitoring --replicas=0

# Restore data
tar -xzf /backup/monitoring/YYYYMMDD/grafana.tar.gz -C /

# Fix permissions
sudo chown -R 472:472 /srv/monitoring_data/*grafana*/

# Start Grafana
kubectl scale deployment grafana -n monitoring --replicas=1
```

### Restore etcd

⚠️ **Warning:** This is destructive. Only for disaster recovery.

```bash
# Stop etcd (stop kube-apiserver first)
systemctl stop etcd

# Restore snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd/snapshot.db \
  --data-dir=/var/lib/etcd-restored

# Replace etcd data directory
mv /var/lib/etcd /var/lib/etcd-old
mv /var/lib/etcd-restored /var/lib/etcd

# Start etcd
systemctl start etcd
```

## Off-site Backup

### Sync to Remote

```bash
rsync -avz /backup/monitoring/ user@remote:/backup/vmstation/monitoring/
```

### Using S3-compatible Storage

```bash
# Install rclone
apt install rclone

# Configure rclone
rclone config

# Sync backups
rclone sync /backup/monitoring remote:vmstation-backups/monitoring
```

## Verification

### Test Restore

Periodically test restore procedures:

1. Create test namespace
2. Restore from backup
3. Verify data integrity
4. Delete test resources

### Backup Integrity Check

```bash
# Verify tar archive
tar -tzf /backup/monitoring/YYYYMMDD/prometheus.tar.gz > /dev/null && echo "OK"
```

## Related Documentation

- [Storage Architecture](../architecture/storage-architecture.md)
- [Day-2 Operations](day-2-operations.md)
- [Disaster Recovery](../troubleshooting/common-issues.md)
