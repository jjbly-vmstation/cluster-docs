# Backup and Restore

This guide covers backup and restore procedures for VMStation.

## What to Backup

| Component | Location | Frequency |
|-----------|----------|-----------|
| Prometheus | /srv/monitoring_data/prometheus | Weekly |
| Loki | /srv/monitoring_data/loki | Weekly |
| Grafana | /srv/monitoring_data/grafana | After changes |
| etcd | /var/lib/etcd | Daily |
| Manifests | Git repository | Automatic |

## Monitoring Data Backup

### Create Backup Directory

```bash
sudo mkdir -p /backup/monitoring
sudo chmod 700 /backup
```

### Backup Prometheus

```bash
# Stop Prometheus (optional, for consistency)
kubectl scale statefulset prometheus -n monitoring --replicas=0

# Create backup
sudo tar -czf /backup/monitoring/prometheus-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/prometheus/

# Restart Prometheus
kubectl scale statefulset prometheus -n monitoring --replicas=1
```

### Backup Loki

```bash
# Stop Loki (optional)
kubectl scale statefulset loki -n monitoring --replicas=0

# Create backup
sudo tar -czf /backup/monitoring/loki-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/loki/

# Restart Loki
kubectl scale statefulset loki -n monitoring --replicas=1
```

### Backup Grafana

```bash
# Stop Grafana (optional)
kubectl scale deployment grafana -n monitoring --replicas=0

# Create backup
sudo tar -czf /backup/monitoring/grafana-$(date +%Y%m%d).tar.gz \
  /srv/monitoring_data/grafana/

# Restart Grafana
kubectl scale deployment grafana -n monitoring --replicas=1
```

### Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/backup-monitoring.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR=/backup/monitoring

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup each component
for component in prometheus loki grafana; do
  tar -czf $BACKUP_DIR/${component}-$DATE.tar.gz \
    /srv/monitoring_data/${component}/ 2>/dev/null
done

# Cleanup old backups (keep 7 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

### Schedule with Cron

```bash
# Add to crontab
0 2 * * * /usr/local/bin/backup-monitoring.sh
```

## etcd Backup

### Create etcd Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%Y%m%d).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### Verify Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-$(date +%Y%m%d).db
```

## Grafana Dashboard Export

### Export via UI

1. Open dashboard in Grafana
2. Click Settings → JSON Model
3. Copy JSON
4. Save to `manifests/monitoring/dashboards/`

### Export via API

```bash
# List dashboards
curl -s http://192.168.4.63:30300/api/search | jq '.[].uid'

# Export dashboard
curl -s http://192.168.4.63:30300/api/dashboards/uid/<uid> | jq '.dashboard' > dashboard.json
```

## Restore Procedures

### Restore Prometheus

```bash
# Stop Prometheus
kubectl scale statefulset prometheus -n monitoring --replicas=0

# Clear existing data
sudo rm -rf /srv/monitoring_data/prometheus/*

# Restore backup
sudo tar -xzf /backup/monitoring/prometheus-20251014.tar.gz -C /

# Fix permissions
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus

# Start Prometheus
kubectl scale statefulset prometheus -n monitoring --replicas=1
```

### Restore Loki

```bash
# Stop Loki
kubectl scale statefulset loki -n monitoring --replicas=0

# Clear existing data
sudo rm -rf /srv/monitoring_data/loki/*

# Restore backup
sudo tar -xzf /backup/monitoring/loki-20251014.tar.gz -C /

# Fix permissions
sudo chown -R 10001:10001 /srv/monitoring_data/loki

# Start Loki
kubectl scale statefulset loki -n monitoring --replicas=1
```

### Restore Grafana

```bash
# Stop Grafana
kubectl scale deployment grafana -n monitoring --replicas=0

# Clear existing data
sudo rm -rf /srv/monitoring_data/grafana/*

# Restore backup
sudo tar -xzf /backup/monitoring/grafana-20251014.tar.gz -C /

# Fix permissions
sudo chown -R 472:472 /srv/monitoring_data/grafana

# Start Grafana
kubectl scale deployment grafana -n monitoring --replicas=1
```

### Restore etcd

```bash
# Stop kube-apiserver (on control plane)
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# Restore snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-20251014.db \
  --data-dir=/var/lib/etcd-restored

# Replace etcd data
mv /var/lib/etcd /var/lib/etcd-old
mv /var/lib/etcd-restored /var/lib/etcd

# Restart etcd
systemctl restart etcd

# Restore kube-apiserver
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

## Disaster Recovery

### Full Cluster Recovery

1. **Rebuild nodes** from scratch
2. **Restore etcd** from snapshot
3. **Redeploy applications** from manifests
4. **Restore monitoring data** from backups

### Minimal Recovery

If etcd is lost but nodes are intact:

```bash
# Reset cluster
./deploy.sh reset

# Redeploy
./deploy.sh kubespray

# Restore monitoring data
# Follow restore procedures above
```

## Verification

### Verify Prometheus Restore

```bash
# Check pod status
kubectl get pods -n monitoring -l app=prometheus

# Check data availability
curl -s http://192.168.4.63:30090/api/v1/query?query=up | jq
```

### Verify Loki Restore

```bash
# Check pod status
kubectl get pods -n monitoring -l app=loki

# Check labels
curl -s http://192.168.4.63:31100/loki/api/v1/labels | jq
```

### Verify Grafana Restore

```bash
# Check pod status
kubectl get pods -n monitoring -l app=grafana

# Check dashboards
curl -s http://192.168.4.63:30300/api/search | jq
```

## Related Documentation

- [Storage Architecture](../architecture/storage-architecture.md)
- [Day-2 Operations](day-2-operations.md)
- [Troubleshooting](../troubleshooting/common-issues.md)
