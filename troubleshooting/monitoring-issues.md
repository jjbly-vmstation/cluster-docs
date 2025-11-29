# Monitoring Issues

This guide covers troubleshooting the VMStation monitoring stack.

## Quick Diagnostics

```bash
# Automated diagnostics
./scripts/diagnose-monitoring-stack.sh

# Automated validation
./scripts/validate-monitoring-stack.sh

# Automated remediation
./scripts/remediate-monitoring-stack.sh
```

## Prometheus Issues

### CrashLoopBackOff

**Symptoms:**
```
prometheus-0   1/2   CrashLoopBackOff   9 restarts   24m
```

**Diagnosis:**
```bash
kubectl logs prometheus-0 -n monitoring -c prometheus --previous
```

**Common Error - Permission Denied:**
```
opening storage failed: lock DB directory: open /prometheus/lock: permission denied
```

**Solution:**

Fix SecurityContext:
```yaml
securityContext:
  runAsUser: 65534
  runAsGroup: 65534  # Add this line
  fsGroup: 65534
```

Or fix permissions:
```bash
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus
kubectl delete pod prometheus-0 -n monitoring
```

### TSDB Corruption

**Symptoms:**
- Pod restarts frequently
- "out of sequence m-mapped chunk" errors

**Solution:**

Prometheus auto-recovers by:
1. Detecting corrupted chunks
2. Discarding bad data
3. Replaying WAL

If persistent:
```bash
# Backup and clear data
kubectl scale statefulset prometheus -n monitoring --replicas=0
sudo rm -rf /srv/monitoring_data/prometheus/*
kubectl scale statefulset prometheus -n monitoring --replicas=1
```

### Targets Down

**Diagnosis:**
```bash
curl -s http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | select(.health!="up")'
```

**Common Issues:**
- Node-exporter not running
- Firewall blocking port 9100
- Node sleeping

**Solution:**
```bash
# Check node-exporter
kubectl get pods -n monitoring -l app=node-exporter -o wide

# Test connectivity
curl http://192.168.4.61:9100/metrics
```

## Loki Issues

### Not Ready

**Symptoms:**
```
loki-0   0/1   Running   4 restarts
```

**Common Error - Connection Refused:**
```
dial tcp 127.0.0.1:9095: connect: connection refused
```

**Solution:**

Disable frontend_worker for single-instance:
```yaml
# Comment out in loki config
# frontend_worker:
#   frontend_address: 127.0.0.1:9095
```

### Parse Errors

**Symptoms:**
```
JSONParserErr: Value looks like object, but can't find closing '}' symbol
```

**Cause:** Malformed JSON from upstream sources

**Solution:**

Use error handling in queries:
```logql
{job="syslog"} | json | __error__=""
```

### Permission Issues

**Error:**
```
error creating bucket: mkdir /loki/chunks: permission denied
```

**Solution:**
```bash
sudo chown -R 10001:10001 /srv/monitoring_data/loki
kubectl delete pod loki-0 -n monitoring
```

### Schema Errors

**Error:**
```
boltdb-shipper requires 24h index period, got 168h
```

**Solution:**

Fix schema_config:
```yaml
schema_config:
  configs:
    - period: 24h  # Must be 24h for boltdb-shipper
```

## Grafana Issues

### Plugin Download Failed

**Error:**
```
Get "https://grafana.com/api/plugins/repo/...": context deadline exceeded
```

**Solution:**
```bash
kubectl set env deployment/grafana -n monitoring GF_INSTALL_PLUGINS-
kubectl rollout restart deployment/grafana -n monitoring
```

### Dashboard Template Error

**Error:**
```
Error updating options: (intermediate value).map is not a function
```

**Solution:**

Add templating section to dashboard JSON:
```json
{
  "schemaVersion": 27,
  "title": "My Dashboard",
  "templating": {
    "list": []
  }
}
```

### Datasource Connection Failed

**Error:**
```
dial tcp: lookup prometheus.monitoring.svc.cluster.local: no such host
```

**Diagnosis:**
```bash
kubectl exec -n monitoring deployment/grafana -- nslookup prometheus.monitoring.svc.cluster.local
```

**Solution:**
```bash
# Restart CoreDNS
kubectl rollout restart deployment/coredns -n kube-system

# Or fix kubelet DNS
ssh <node> "sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml"
ssh <node> "systemctl restart kubelet"
```

## Promtail Issues

### Not Shipping Logs

**Diagnosis:**
```bash
kubectl logs -n monitoring -l app=promtail
```

**Common Issues:**
- Wrong Loki URL
- Permission to read logs
- Loki not ready

**Solution:**
```bash
# Check config
kubectl get configmap promtail-config -n monitoring -o yaml

# Restart
kubectl rollout restart daemonset/promtail -n monitoring
```

## Node Exporter Issues

### Metrics Missing

**Diagnosis:**
```bash
# Test locally
curl http://192.168.4.61:9100/metrics

# Check pod
kubectl get pods -n monitoring -l app=node-exporter -o wide
```

**Solutions:**
```bash
# Restart daemonset
kubectl rollout restart daemonset/node-exporter -n monitoring

# Check hostNetwork
kubectl get pods -n monitoring -l app=node-exporter -o yaml | grep hostNetwork
```

## Empty Endpoints

**Symptoms:**
```
kubectl get endpoints prometheus -n monitoring
# Shows: <none>
```

**Cause:** Pods not Ready

**Solution:**

Fix underlying pod issues first. Endpoints populate automatically when pods are Ready.

## Validation Commands

```bash
# Health checks
curl http://192.168.4.63:30090/-/healthy  # Prometheus
curl http://192.168.4.63:31100/ready       # Loki
curl http://192.168.4.63:30300/api/health  # Grafana

# Check all pods
kubectl get pods -n monitoring

# Check endpoints
kubectl get endpoints -n monitoring
```

## Related Documentation

- [Common Issues](common-issues.md)
- [Grafana Fixes](grafana-fixes.md)
- [Diagnostic Procedures](diagnostic-procedures.md)
- [Monitoring Architecture](../architecture/monitoring-architecture.md)
