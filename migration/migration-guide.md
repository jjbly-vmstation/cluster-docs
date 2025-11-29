# Migration Guide

Procedures for migrating VMStation components.

## Cluster Migration

### Upgrading Kubernetes Version

#### Using Kubespray

1. Update version in configuration
2. Run upgrade playbook
3. Verify cluster

```bash
# Edit k8s-cluster.yml
kube_version: v1.30.0

# Run upgrade
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini upgrade-cluster.yml -b
```

#### Using kubeadm

1. Upgrade control plane
2. Upgrade workers
3. Verify

See [Upgrades](../operations/upgrades.md) for details.

### Migrating to New Node

1. Drain old node
2. Add new node to inventory
3. Run deployment
4. Remove old node

```bash
# Drain old node
kubectl drain <old-node> --ignore-daemonsets --delete-emptydir-data

# Deploy to new node
./deploy.sh debian

# Remove old node
kubectl delete node <old-node>
```

## Data Migration

### Prometheus Data

```bash
# Stop Prometheus
kubectl scale statefulset prometheus -n monitoring --replicas=0

# Copy data
rsync -avz /srv/monitoring_data/prometheus/ user@newhost:/srv/monitoring_data/prometheus/

# Update PV to point to new location
# Start Prometheus
kubectl scale statefulset prometheus -n monitoring --replicas=1
```

### Loki Data

```bash
# Stop Loki
kubectl scale statefulset loki -n monitoring --replicas=0

# Copy data
rsync -avz /srv/monitoring_data/loki/ user@newhost:/srv/monitoring_data/loki/

# Start Loki
kubectl scale statefulset loki -n monitoring --replicas=1
```

### Grafana Data

```bash
# Stop Grafana
kubectl scale deployment grafana -n monitoring --replicas=0

# Copy data
rsync -avz /srv/monitoring_data/grafana/ user@newhost:/srv/monitoring_data/grafana/

# Start Grafana
kubectl scale deployment grafana -n monitoring --replicas=1
```

## Application Migration

### Between Nodes

1. Update node selector in manifest
2. Apply updated manifest
3. Pod reschedules automatically

### To New Cluster

1. Export manifests
2. Apply to new cluster
3. Migrate data (if applicable)

## Configuration Migration

### Inventory

1. Export current inventory
2. Update host details
3. Test connectivity

### Manifests

1. Copy manifests to new location
2. Update cluster-specific values
3. Apply to new cluster

## Rollback Procedures

### Cluster Rollback

If upgrade fails:

1. Stop problematic services
2. Restore etcd from backup
3. Restart services

### Data Rollback

1. Stop services
2. Restore from backup
3. Fix permissions
4. Start services

## Related Documentation

- [Backup & Restore](../operations/backup-restore.md)
- [Upgrades](../operations/upgrades.md)
