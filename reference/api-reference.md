# API Reference

API endpoints for VMStation services.

## Prometheus API

Base URL: `http://192.168.4.63:30090`

### Health

```bash
# Health check
curl http://192.168.4.63:30090/-/healthy

# Ready check
curl http://192.168.4.63:30090/-/ready
```

### Query

```bash
# Instant query
curl 'http://192.168.4.63:30090/api/v1/query?query=up'

# Range query
curl 'http://192.168.4.63:30090/api/v1/query_range?query=up&start=2024-01-01T00:00:00Z&end=2024-01-01T01:00:00Z&step=15s'
```

### Targets

```bash
# Get all targets
curl http://192.168.4.63:30090/api/v1/targets

# Filter targets
curl http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | select(.health=="up")'
```

### Labels

```bash
# Get all labels
curl http://192.168.4.63:30090/api/v1/labels

# Get label values
curl http://192.168.4.63:30090/api/v1/label/job/values
```

### Admin

```bash
# Reload configuration
curl -X POST http://192.168.4.63:30090/-/reload

# Take TSDB snapshot
curl -X POST http://192.168.4.63:30090/api/v1/admin/tsdb/snapshot
```

## Loki API

Base URL: `http://192.168.4.63:31100`

### Health

```bash
# Ready check
curl http://192.168.4.63:31100/ready

# Metrics
curl http://192.168.4.63:31100/metrics
```

### Query

```bash
# Query logs
curl -G 'http://192.168.4.63:31100/loki/api/v1/query' \
  --data-urlencode 'query={namespace="monitoring"}'

# Range query
curl -G 'http://192.168.4.63:31100/loki/api/v1/query_range' \
  --data-urlencode 'query={namespace="monitoring"}' \
  --data-urlencode 'start=2024-01-01T00:00:00Z' \
  --data-urlencode 'end=2024-01-01T01:00:00Z'
```

### Labels

```bash
# Get all labels
curl http://192.168.4.63:31100/loki/api/v1/labels

# Get label values
curl http://192.168.4.63:31100/loki/api/v1/label/namespace/values
```

### Push

```bash
# Push logs (requires proper format)
curl -X POST http://192.168.4.63:31100/loki/api/v1/push \
  -H "Content-Type: application/json" \
  -d '{"streams": [{"stream": {"job": "test"}, "values": [["1234567890000000000", "test log line"]]}]}'
```

## Grafana API

Base URL: `http://192.168.4.63:30300`

### Health

```bash
# Health check
curl http://192.168.4.63:30300/api/health
```

### Datasources

```bash
# List datasources
curl http://192.168.4.63:30300/api/datasources

# Get datasource by ID
curl http://192.168.4.63:30300/api/datasources/1

# Test datasource
curl -X POST http://192.168.4.63:30300/api/datasources/1/health
```

### Dashboards

```bash
# Search dashboards
curl http://192.168.4.63:30300/api/search

# Get dashboard by UID
curl http://192.168.4.63:30300/api/dashboards/uid/<uid>

# Create/update dashboard
curl -X POST http://192.168.4.63:30300/api/dashboards/db \
  -H "Content-Type: application/json" \
  -d '{"dashboard": {...}, "overwrite": true}'
```

### Users

```bash
# Current user
curl http://192.168.4.63:30300/api/user

# List users (admin)
curl -u admin:password http://192.168.4.63:30300/api/users
```

## Kubernetes API

Base URL: `https://192.168.4.63:6443`

### Authentication

```bash
# Using kubeconfig
kubectl --kubeconfig=/etc/kubernetes/admin.conf api-versions

# Direct API (with token)
TOKEN=$(kubectl get secret -n kube-system $(kubectl get sa -n kube-system default -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d)

curl -k -H "Authorization: Bearer $TOKEN" https://192.168.4.63:6443/api/v1/namespaces
```

### Common Endpoints

```bash
# API versions
GET /api
GET /apis

# Namespaces
GET /api/v1/namespaces
GET /api/v1/namespaces/{namespace}

# Pods
GET /api/v1/namespaces/{namespace}/pods
GET /api/v1/namespaces/{namespace}/pods/{name}
GET /api/v1/namespaces/{namespace}/pods/{name}/log

# Services
GET /api/v1/namespaces/{namespace}/services

# Deployments
GET /apis/apps/v1/namespaces/{namespace}/deployments
```

## Related Documentation

- [CLI Reference](cli-reference.md)
- [Prometheus](../components/monitoring/prometheus.md)
- [Grafana](../components/monitoring/grafana.md)
- [Loki](../components/monitoring/loki.md)
