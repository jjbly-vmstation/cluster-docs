# Grafana Fixes

Grafana-specific troubleshooting.

## Common Issues

### Plugin Installation Timeout

**Symptom:**
```
Get "https://grafana.com/api/plugins/repo/grafana-kubernetes-app": context deadline exceeded
```

Grafana fails to start due to plugin download timeout.

**Fix:**
```bash
kubectl set env deployment/grafana -n monitoring GF_INSTALL_PLUGINS-
kubectl rollout restart deployment/grafana -n monitoring
```

### Dashboard Templating Error

**Symptom:**
```
Error updating options: (intermediate value).map is not a function
```

Template variables fail to load.

**Cause:** Missing `templating` section in dashboard JSON

**Fix:** Add templating section:
```json
{
  "schemaVersion": 27,
  "title": "Dashboard Name",
  "templating": {
    "list": []
  },
  "panels": [...]
}
```

Update ConfigMap and restart:
```bash
kubectl apply -f manifests/monitoring/grafana.yaml
kubectl rollout restart deployment/grafana -n monitoring
```

### Datasource Not Working

**Symptom:** "Data source is not working"

**Check connectivity:**
```bash
kubectl exec -n monitoring $(kubectl get pod -n monitoring -l app=grafana -o jsonpath='{.items[0].metadata.name}') -- wget -O- http://prometheus:9090/-/healthy
```

**Check DNS:**
```bash
kubectl exec -n monitoring $(kubectl get pod -n monitoring -l app=grafana -o jsonpath='{.items[0].metadata.name}') -- nslookup prometheus.monitoring.svc.cluster.local
```

**Fix DNS issues:**
```bash
# If nodelocaldns conflicts
kubectl delete daemonset nodelocaldns -n kube-system

# Update kubelet
sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml
systemctl restart kubelet

# Restart Grafana
kubectl rollout restart deployment/grafana -n monitoring
```

### Dashboard JSON Parse Error

**Symptom:**
```
logger=provisioning.dashboard error="invalid character '{'"
```

**Cause:** Malformed JSON in dashboard file

**Fix:**
1. Export dashboard from ConfigMap
2. Validate JSON: `jq . < dashboard.json`
3. Fix errors
4. Update ConfigMap

### Syslog Parse Errors

**Symptom:**
```
JSONParserErr: Value looks like object, but can't find closing '}'
```

**Cause:** Incomplete JSON from syslog sources

**Fix:** Queries already handle this:
```logql
{job="syslog"} | json | __error__=""
```

## Configuration

### Anonymous Access

Enable anonymous viewing:
```yaml
env:
  - name: GF_AUTH_ANONYMOUS_ENABLED
    value: "true"
  - name: GF_AUTH_ANONYMOUS_ORG_ROLE
    value: "Viewer"
```

### Admin Password

Change admin password:
```yaml
env:
  - name: GF_SECURITY_ADMIN_PASSWORD
    value: "newpassword"
```

Or via UI: Admin → Change Password

### Add Datasource

Via UI:
1. Configuration → Data Sources
2. Add data source
3. Select type (Prometheus/Loki)
4. Configure URL
5. Save & Test

Via provisioning:
```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    access: proxy
    isDefault: true
```

## Dashboard Management

### Import Dashboard

1. Go to Dashboards → Import
2. Enter dashboard ID or upload JSON
3. Select datasource
4. Import

### Export Dashboard

1. Go to Dashboard
2. Settings → JSON Model
3. Copy JSON

### Backup Dashboards

```bash
# Get all dashboard UIDs
curl -s http://192.168.4.63:30300/api/search | jq '.[].uid'

# Export each dashboard
for uid in $(curl -s http://192.168.4.63:30300/api/search | jq -r '.[].uid'); do
  curl -s "http://192.168.4.63:30300/api/dashboards/uid/$uid" > "dashboard-$uid.json"
done
```

## Troubleshooting Checklist

1. **Pod running?**
   ```bash
   kubectl get pods -n monitoring -l app=grafana
   ```

2. **Logs show errors?**
   ```bash
   kubectl logs -n monitoring -l app=grafana
   ```

3. **PVC bound?**
   ```bash
   kubectl get pvc -n monitoring | grep grafana
   ```

4. **Service accessible?**
   ```bash
   curl http://192.168.4.63:30300/api/health
   ```

5. **Datasources connected?**
   - UI: Configuration → Data Sources → Test

## Recovery

### Restart Grafana

```bash
kubectl rollout restart deployment/grafana -n monitoring
```

### Delete and Recreate Pod

```bash
kubectl delete pod -n monitoring -l app=grafana
```

### Redeploy

```bash
kubectl delete deployment grafana -n monitoring
kubectl apply -f manifests/monitoring/grafana.yaml
```

### Fresh Start

```bash
# Delete PVC to reset data
kubectl delete pvc grafana-pvc -n monitoring
kubectl apply -f manifests/monitoring/grafana.yaml
```

## Related Documentation

- [Monitoring Issues](monitoring-issues.md)
- [Grafana Component](../components/monitoring/grafana.md)
- [Dashboards](../components/monitoring/dashboards.md)
