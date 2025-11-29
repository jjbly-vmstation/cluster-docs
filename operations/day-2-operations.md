# Day-2 Operations

Routine operational tasks for VMStation.

## Daily Tasks

### Health Check

```bash
# Quick status
kubectl get nodes
kubectl get pods -A | grep -v Running

# Detailed validation
./tests/test-complete-validation.sh
```

### Monitor Dashboards

- Grafana: http://192.168.4.63:30300
- Check node metrics
- Review log errors

## Weekly Tasks

### Review Alerts

Check for persistent issues in:
- Prometheus alerts
- Grafana annotations
- Node health trends

### Storage Usage

```bash
# Check PVC usage
kubectl get pvc -n monitoring

# Check disk usage on masternode
df -h /srv/monitoring_data
du -sh /srv/monitoring_data/pvc-*
```

### Log Review

```bash
# Check for errors in Loki
# Grafana → Explore → Loki
# Query: {namespace="monitoring"} |= "error"
```

## Monthly Tasks

### Backup Verification

```bash
# Verify backup scripts run
ls -la /backup/monitoring/

# Test restore procedure (on test env)
```

### Certificate Expiry

```bash
# Check Kubernetes certificates
kubeadm certs check-expiration
```

### Security Updates

```bash
# Update nodes
apt update && apt upgrade -y
```

## Node Management

### Check Node Status

```bash
kubectl get nodes -o wide
kubectl describe node <nodename>
```

### Cordon Node (Maintenance)

```bash
kubectl cordon <nodename>
kubectl drain <nodename> --ignore-daemonsets --delete-emptydir-data
```

### Uncordon Node

```bash
kubectl uncordon <nodename>
```

## Pod Management

### Check Pod Status

```bash
kubectl get pods -A
kubectl describe pod <podname> -n <namespace>
```

### View Logs

```bash
kubectl logs <podname> -n <namespace>
kubectl logs <podname> -n <namespace> --previous
kubectl logs -f <podname> -n <namespace>  # Follow
```

### Restart Pod

```bash
kubectl delete pod <podname> -n <namespace>
# Pod recreates automatically
```

### Restart Deployment

```bash
kubectl rollout restart deployment <name> -n <namespace>
```

## Service Management

### Check Services

```bash
kubectl get svc -A
kubectl get endpoints -A
```

### Test Service

```bash
curl http://192.168.4.63:<nodeport>
```

## Resource Monitoring

### Pod Resources

```bash
kubectl top pods -A
kubectl top nodes
```

### Prometheus Queries

```promql
# Node CPU usage
100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Node memory usage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk usage
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100
```

## Troubleshooting Workflow

1. Check node status: `kubectl get nodes`
2. Check pod status: `kubectl get pods -A`
3. Check events: `kubectl get events --sort-by='.lastTimestamp'`
4. Check logs: `kubectl logs <pod> -n <ns>`
5. Run diagnostics: `./scripts/diagnose-monitoring-stack.sh`

## Related Documentation

- [Cluster Management](cluster-management.md)
- [Troubleshooting](../troubleshooting/README.md)
- [Power Management](power-management.md)
