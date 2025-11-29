# Day-2 Operations

This guide covers routine operational tasks after initial deployment.

## Daily Operations

### Health Checks

```bash
# Check node status
kubectl get nodes

# Check all pods
kubectl get pods -A | grep -v Running | grep -v Completed

# Check monitoring
curl -s http://192.168.4.63:30090/-/healthy
curl -s http://192.168.4.63:30300/api/health | jq
```

### View Logs

```bash
# View pod logs
kubectl logs <pod-name> -n <namespace>

# Follow logs
kubectl logs -f <pod-name> -n <namespace>

# In Grafana
# Navigate to Explore → Select Loki
# Query: {namespace="monitoring"}
```

### Monitor Resources

```bash
# Node resources
kubectl top nodes

# Pod resources
kubectl top pods -A
```

## Weekly Operations

### Review Alerts

1. Open Grafana → Alerting
2. Review active alerts
3. Address any critical issues

### Check Storage Usage

```bash
# Disk usage
df -h /srv/monitoring_data

# Per PVC
kubectl get pvc -A
```

### Verify Backups

```bash
ls -lh /backup/monitoring/
```

## Routine Maintenance

### Clean Up Old Data

Prometheus and Loki automatically manage retention:
- Prometheus: 15 days
- Loki: 31 days

### Update Manifests

```bash
cd /srv/monitoring_data/VMStation
git pull
kubectl apply -f manifests/monitoring/
```

### Restart Services

```bash
# Rolling restart
kubectl rollout restart deployment/grafana -n monitoring
kubectl rollout restart statefulset/prometheus -n monitoring
kubectl rollout restart statefulset/loki -n monitoring
```

## Common Tasks

### Add Application

See [Application Deployment](../deployment/application-deployment.md)

### Scale Deployment

```bash
kubectl scale deployment <name> --replicas=3 -n <namespace>
```

### Update Image

```bash
kubectl set image deployment/<name> <container>=<image>:<tag> -n <namespace>
```

### Drain Node for Maintenance

```bash
# Cordon (prevent scheduling)
kubectl cordon <node>

# Drain (evict pods)
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data

# After maintenance
kubectl uncordon <node>
```

### View Events

```bash
# Cluster events
kubectl get events -A --sort-by='.lastTimestamp'

# Namespace events
kubectl get events -n monitoring
```

## Monitoring Operations

### Check Prometheus Targets

```bash
curl -s http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'
```

### Query Metrics

```bash
# Current CPU usage
curl -s 'http://192.168.4.63:30090/api/v1/query?query=1-avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))by(node)' | jq

# Memory usage
curl -s 'http://192.168.4.63:30090/api/v1/query?query=node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes' | jq
```

### Query Logs

```bash
# Recent errors
curl -s 'http://192.168.4.63:31100/loki/api/v1/query_range?query={namespace="monitoring"}|="error"&limit=100'
```

## Automation

### Scheduled Tasks

VMStation includes automated tasks:
- Auto-sleep monitoring (hourly)
- Log rotation (daily)
- Prometheus compaction (automatic)

### Validation Scripts

```bash
# Complete validation
./tests/test-complete-validation.sh

# Monitoring validation
./scripts/validate-monitoring-stack.sh

# Time sync validation
./tests/validate-time-sync.sh
```

## Incident Response

### Pod CrashLoopBackOff

```bash
# Get pod details
kubectl describe pod <pod> -n <namespace>

# Check logs
kubectl logs <pod> -n <namespace> --previous
```

### Node NotReady

```bash
# Check node status
kubectl describe node <node>

# Check kubelet
ssh <node> "systemctl status kubelet"
ssh <node> "journalctl -u kubelet -n 50"
```

### Service Unavailable

```bash
# Check endpoints
kubectl get endpoints <service> -n <namespace>

# Check service
kubectl describe svc <service> -n <namespace>
```

## Documentation

Keep operational notes:
- Issues encountered
- Resolutions applied
- Configuration changes
- Performance observations

## Related Documentation

- [Cluster Management](cluster-management.md)
- [Troubleshooting](../troubleshooting/common-issues.md)
- [Power Management](power-management.md)
