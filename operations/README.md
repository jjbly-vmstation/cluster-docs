# Operations

This section covers day-to-day operations, maintenance, and management of VMStation clusters.

## Documents

- [Day-2 Operations](day-2-operations.md) - Ongoing operational tasks
- [Cluster Management](cluster-management.md) - Managing Kubernetes clusters
- [Backup & Restore](backup-restore.md) - Data protection procedures
- [Scaling](scaling.md) - Scaling your infrastructure
- [Upgrades](upgrades.md) - Upgrade procedures
- [Power Management](power-management.md) - Auto-sleep and Wake-on-LAN

## Quick Reference

### Common Commands

```bash
# Cluster status
kubectl get nodes
kubectl get pods -A

# Monitoring status
kubectl get pods -n monitoring

# Validate stack
./scripts/validate-monitoring-stack.sh
```

### Access URLs

| Service | URL |
|---------|-----|
| Grafana | http://192.168.4.63:30300 |
| Prometheus | http://192.168.4.63:30090 |
| Loki | http://192.168.4.63:31100 |

### Wake Sleeping Nodes

```bash
wakeonlan b8:ac:6f:7e:6c:9d  # storagenodet3500
wakeonlan d0:94:66:30:d6:63  # homelab
```
