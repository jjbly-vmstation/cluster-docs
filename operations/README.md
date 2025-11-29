# Operations

Day-to-day operations and management of VMStation.

## Documents

- [Day-2 Operations](day-2-operations.md) - Routine operations
- [Cluster Management](cluster-management.md) - Managing nodes and workloads
- [Backup & Restore](backup-restore.md) - Data protection
- [Scaling](scaling.md) - Adding/removing capacity
- [Upgrades](upgrades.md) - Version upgrades
- [Power Management](power-management.md) - Auto-sleep and Wake-on-LAN

## Quick Reference

### Cluster Status

```bash
kubectl get nodes
kubectl get pods -A
```

### Monitoring Status

```bash
kubectl get pods -n monitoring
./scripts/validate-monitoring-stack.sh
```

### Service Access

| Service | URL |
|---------|-----|
| Grafana | http://192.168.4.63:30300 |
| Prometheus | http://192.168.4.63:30090 |
| Loki | http://192.168.4.63:31100 |

## Common Tasks

### Check Cluster Health

```bash
./tests/test-complete-validation.sh
```

### Wake Sleeping Nodes

```bash
wakeonlan b8:ac:6f:7e:6c:9d  # storagenodet3500
wakeonlan d0:94:66:30:d6:63  # homelab
```

### Restart Monitoring

```bash
kubectl rollout restart statefulset prometheus -n monitoring
kubectl rollout restart statefulset loki -n monitoring
kubectl rollout restart deployment grafana -n monitoring
```

## Related Documentation

- [Troubleshooting](../troubleshooting/README.md)
- [Architecture](../architecture/README.md)
- [Deployment](../deployment/README.md)
