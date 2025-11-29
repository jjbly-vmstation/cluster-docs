# Grafana Fixes

This guide covers Grafana-specific troubleshooting and fixes.

## Common Issues

### Plugin Installation Failed

**Symptoms:**
```
Get "https://grafana.com/api/plugins/repo/grafana-kubernetes-app": context deadline exceeded
```

Pod in CrashLoopBackOff due to plugin download timeout.

**Solution:**

Remove plugin environment variable:
```bash
kubectl set env deployment/grafana -n monitoring GF_INSTALL_PLUGINS-
kubectl rollout restart deployment/grafana -n monitoring
```

### Dashboard Template Error

**Symptoms:**
```
Error updating options: (intermediate value).map is not a function
```

**Cause:** Missing `templating` section in dashboard JSON

**Solution:**

Add templating section:
```json
{
  "schemaVersion": 27,
  "title": "My Dashboard",
  "templating": {
    "list": []
  },
  "panels": [...]
}
```

### Datasource Not Working

**Symptoms:**
- "Data source is not working"
- Empty panels

**Diagnosis:**
```bash
# Check datasource config
curl -s http://192.168.4.63:30300/api/datasources | jq

# Test Prometheus directly
curl -s http://192.168.4.63:30090/-/healthy

# Test Loki directly
curl -s http://192.168.4.63:31100/ready
```

**Solution:**

Check datasource URLs in Grafana:
- Prometheus: `http://prometheus:9090`
- Loki: `http://loki:3100`

### DNS Resolution Failed

**Symptoms:**
```
dial tcp: lookup prometheus.monitoring.svc.cluster.local: no such host
```

**Solution:**
```bash
# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Restart CoreDNS
kubectl rollout restart deployment/coredns -n kube-system

# Restart Grafana
kubectl rollout restart deployment/grafana -n monitoring
```

### Anonymous Access Not Working

**Check Configuration:**
```bash
kubectl get configmap grafana-config -n monitoring -o yaml | grep -A5 auth.anonymous
```

**Expected Config:**
```ini
[auth.anonymous]
enabled = true
org_name = Main Org.
org_role = Viewer
```

**Apply Fix:**
```bash
kubectl rollout restart deployment/grafana -n monitoring
```

### Dashboard JSON Parsing Error

**Symptoms:**
```
invalid character '{' looking for beginning of object key string
```

**Cause:** Malformed JSON in dashboard ConfigMap

**Solution:**

1. Extract dashboard:
```bash
kubectl get configmap grafana-dashboards -n monitoring -o yaml > dashboards.yaml
```

2. Validate JSON:
```bash
cat dashboards.yaml | grep -A999 "syslog-dashboard.json" | python3 -m json.tool
```

3. Fix JSON syntax errors

4. Reapply:
```bash
kubectl apply -f dashboards.yaml
kubectl rollout restart deployment/grafana -n monitoring
```

## Grafana Configuration

### Access Grafana API

```bash
# List datasources
curl -s http://192.168.4.63:30300/api/datasources | jq

# List dashboards
curl -s http://192.168.4.63:30300/api/search | jq

# Get dashboard by UID
curl -s http://192.168.4.63:30300/api/dashboards/uid/<uid> | jq
```

### Reset Admin Password

```bash
kubectl exec -n monitoring deployment/grafana -- grafana-cli admin reset-admin-password newpassword
```

### Export Dashboard

```bash
curl -s http://192.168.4.63:30300/api/dashboards/uid/<uid> | jq '.dashboard' > dashboard.json
```

### Import Dashboard

```bash
curl -X POST -H "Content-Type: application/json" \
  -d @dashboard.json \
  http://192.168.4.63:30300/api/dashboards/db
```

## Dashboard Best Practices

### Include Templating

Always include templating section:
```json
{
  "templating": {
    "list": []
  }
}
```

### Use Variables

```json
{
  "templating": {
    "list": [
      {
        "name": "namespace",
        "type": "query",
        "datasource": "Prometheus",
        "query": "label_values(namespace)"
      }
    ]
  }
}
```

### Set Refresh

```json
{
  "refresh": "30s",
  "schemaVersion": 27
}
```

## Prometheus Datasource

### Configuration

```json
{
  "name": "Prometheus",
  "type": "prometheus",
  "url": "http://prometheus:9090",
  "access": "proxy",
  "isDefault": true
}
```

### Test Query

```bash
curl -s 'http://192.168.4.63:30300/api/datasources/proxy/1/api/v1/query?query=up'
```

## Loki Datasource

### Configuration

```json
{
  "name": "Loki",
  "type": "loki",
  "url": "http://loki:3100",
  "access": "proxy"
}
```

### Test Query

```bash
curl -s 'http://192.168.4.63:30300/api/datasources/proxy/2/loki/api/v1/labels'
```

## Performance Issues

### Slow Dashboard Loading

1. Reduce query range
2. Add query caching
3. Optimize PromQL queries

### High Memory Usage

1. Reduce concurrent users
2. Limit dashboard auto-refresh
3. Optimize panel queries

## Logs

### View Grafana Logs

```bash
kubectl logs -n monitoring deployment/grafana
```

### Common Log Messages

```
lvl=warn msg="falling back to legacy dashboard provisioning format"
# Not critical - can be ignored

lvl=eror msg="Failed to start provisioning"
# Check dashboard JSON syntax
```

## Related Documentation

- [Monitoring Issues](monitoring-issues.md)
- [Monitoring Architecture](../architecture/monitoring-architecture.md)
- [Dashboards](../components/monitoring/dashboards.md)
