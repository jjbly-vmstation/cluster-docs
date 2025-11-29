# Monitoring Issues

Troubleshooting Prometheus, Grafana, and Loki.

## Diagnostic Tools

```bash
# Run comprehensive diagnostics
./scripts/diagnose-monitoring-stack.sh

# Run validation
./scripts/validate-monitoring-stack.sh

# Run remediation
./scripts/remediate-monitoring-stack.sh
```

## Prometheus Issues

### Prometheus CrashLoopBackOff

**Symptom:**
```
prometheus-0   1/2   CrashLoopBackOff   9 restarts
```

**Check logs:**
```bash
kubectl logs prometheus-0 -n monitoring -c prometheus
```

**Common causes:**

1. **Permission denied (storage)**
   ```
   Error: opening storage failed: lock DB directory: permission denied
   ```
   
   **Fix:** Add `runAsGroup` to SecurityContext:
   ```yaml
   securityContext:
     runAsUser: 65534
     runAsGroup: 65534  # Add this
     fsGroup: 65534
   ```
   
   Fix permissions:
   ```bash
   sudo chown -R 65534:65534 /srv/monitoring_data/*prometheus*/
   ```

2. **TSDB corruption**
   - Prometheus will auto-recover
   - Check logs for WAL replay
   - May take time to start

### Prometheus Targets Down

**Check targets:**
```bash
curl http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job, instance, health}'
```

**Common causes:**

1. **Exporter not running**
   ```bash
   kubectl get pods -n monitoring -l app=node-exporter
   ```

2. **Firewall blocking**
   ```bash
   curl http://<node>:9100/metrics
   ```

3. **Wrong scrape config**
   ```bash
   kubectl get configmap prometheus-config -n monitoring -o yaml
   ```

## Loki Issues

### Loki CrashLoopBackOff

**Symptom:**
```
loki-0   0/1   CrashLoopBackOff
```

**Check logs:**
```bash
kubectl logs loki-0 -n monitoring
```

**Common causes:**

1. **Connection refused (frontend_worker)**
   ```
   Error: dial tcp 127.0.0.1:9095: connection refused
   ```
   
   **Fix:** Disable frontend_worker for single-instance:
   ```yaml
   # Comment out in loki config:
   # frontend_worker:
   #   frontend_address: 127.0.0.1:9095
   ```

2. **Schema validation error**
   ```
   Error: boltdb-shipper requires 24h index period
   ```
   
   **Fix:** Change period to 24h:
   ```yaml
   schema_config:
     configs:
       - period: 24h  # Not 168h
   ```

3. **Permission denied**
   ```bash
   sudo chown -R 10001:10001 /srv/monitoring_data/*loki*/
   ```

### No Logs in Loki

**Check Promtail:**
```bash
kubectl get pods -n monitoring -l app=promtail
kubectl logs -n monitoring -l app=promtail
```

**Test Loki:**
```bash
curl http://192.168.4.63:31100/ready
curl http://192.168.4.63:31100/loki/api/v1/labels
```

## Grafana Issues

### Grafana Not Loading

**Check pod:**
```bash
kubectl get pods -n monitoring -l app=grafana
kubectl logs -n monitoring -l app=grafana
```

**Test endpoint:**
```bash
curl http://192.168.4.63:30300/api/health
```

### Datasource Connection Failed

**Symptom:** "Data source is not working"

**Check endpoints:**
```bash
kubectl get endpoints prometheus loki -n monitoring
```

**From Grafana pod:**
```bash
kubectl exec -n monitoring -l app=grafana -- wget -O- http://prometheus:9090/-/healthy
kubectl exec -n monitoring -l app=grafana -- wget -O- http://loki:3100/ready
```

**Fix DNS:** If using nodelocaldns:
```bash
kubectl delete daemonset nodelocaldns -n kube-system
sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml
systemctl restart kubelet
```

### Dashboard Errors

**Symptom:** "Error updating options: map is not a function"

**Cause:** Missing `templating` section in dashboard JSON

**Fix:**
```json
{
  "templating": {
    "list": []
  }
}
```

### Plugin Installation Failed

**Symptom:** Grafana CrashLoopBackOff, plugin timeout

**Fix:** Remove plugin environment variable:
```bash
kubectl set env deployment/grafana -n monitoring GF_INSTALL_PLUGINS-
```

## Service Endpoints Empty

**Symptom:**
```bash
kubectl get endpoints -n monitoring
# Shows <none> for prometheus/loki
```

**Cause:** Pods are not Ready

**Fix:** Resolve pod issues first (CrashLoopBackOff)

## Validation Checks

### Run Full Validation

```bash
./scripts/validate-monitoring-stack.sh
```

**Checks performed:**
1. Pod Status - Running and Ready
2. Service Endpoints - Populated
3. PVC/PV Bindings - Bound
4. Health Endpoints - HTTP OK
5. DNS Resolution - Working
6. Container Restarts - Low count
7. Log Analysis - No critical errors

## Recovery Procedures

### Restart All Monitoring

```bash
kubectl rollout restart statefulset prometheus -n monitoring
kubectl rollout restart statefulset loki -n monitoring
kubectl rollout restart deployment grafana -n monitoring
kubectl rollout restart daemonset promtail -n monitoring
kubectl rollout restart daemonset node-exporter -n monitoring
```

### Redeploy Monitoring

```bash
./deploy.sh monitoring
```

### Full Reset

```bash
kubectl delete namespace monitoring
./deploy.sh monitoring
```

## Related Documentation

- [Grafana Fixes](grafana-fixes.md)
- [Monitoring Architecture](../architecture/monitoring-architecture.md)
- [Monitoring Deployment](../deployment/monitoring-deployment.md)
