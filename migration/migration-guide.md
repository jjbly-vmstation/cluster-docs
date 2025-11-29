# Migration Guide

Procedures for migrating VMStation.

## Version Migration

### Pre-Migration Checklist

- [ ] Backup current configuration
- [ ] Backup monitoring data
- [ ] Document current state
- [ ] Test in staging if possible

### Backup Procedures

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Backup monitoring data
tar -czf /backup/monitoring-data.tar.gz /srv/monitoring_data/

# Backup manifests
cp -r manifests/ /backup/manifests/
```

### Migration Steps

1. **Stop workloads**
   ```bash
   kubectl scale deployment --all --replicas=0 -n default
   ```

2. **Apply new configuration**
   ```bash
   git pull origin main
   ./deploy.sh monitoring
   ```

3. **Verify**
   ```bash
   ./scripts/validate-monitoring-stack.sh
   ```

4. **Restore workloads**
   ```bash
   kubectl scale deployment --all --replicas=1 -n default
   ```

## Component Migrations

### kubeadm to Kubespray

1. **Backup current cluster**
2. **Run preflight on nodes**
   ```bash
   ansible-playbook -i ansible/inventory/hosts.yml \
     ansible/playbooks/run-preflight-rhel10.yml
   ```
3. **Deploy via Kubespray**
   ```bash
   ./scripts/run-kubespray.sh
   ```
4. **Migrate workloads**
5. **Validate**

### Monitoring Version Upgrade

1. **Backup data**
2. **Update manifests with new versions**
3. **Apply updates**
   ```bash
   kubectl apply -f manifests/monitoring/
   ```
4. **Restart pods**
   ```bash
   kubectl rollout restart statefulset prometheus -n monitoring
   ```
5. **Validate**

## Rollback Procedures

### Quick Rollback

```bash
# Restore from backup
tar -xzf /backup/monitoring-data.tar.gz -C /

# Apply previous manifests
kubectl apply -f /backup/manifests/monitoring/

# Restart pods
kubectl rollout restart -n monitoring statefulset prometheus loki
```

### etcd Restore

⚠️ **Destructive - for disaster recovery only**

```bash
systemctl stop etcd

ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restored

mv /var/lib/etcd /var/lib/etcd-old
mv /var/lib/etcd-restored /var/lib/etcd

systemctl start etcd
```

## Data Migration

### Prometheus Data

Prometheus TSDB is not portable between versions. Plan for:
- Historical data loss during major upgrades
- Use Thanos for long-term storage

### Grafana Dashboards

Export and import dashboards:

```bash
# Export
curl http://192.168.4.63:30300/api/dashboards/uid/<uid> > dashboard.json

# Import
curl -X POST http://192.168.4.63:30300/api/dashboards/db \
  -H "Content-Type: application/json" \
  -d @dashboard.json
```

### Loki Data

Loki chunks should be compatible within minor versions.

## Related Documentation

- [Backup & Restore](../operations/backup-restore.md)
- [Upgrades](../operations/upgrades.md)
- [Monorepo to Modular](monorepo-to-modular.md)
