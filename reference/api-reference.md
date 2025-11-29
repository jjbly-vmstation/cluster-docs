# API Reference

API endpoints for VMStation services.

## Prometheus API

Base URL: `http://192.168.4.63:30090`

### Health

```bash
# Health check
GET /-/healthy

# Ready check
GET /-/ready
```

### Query

```bash
# Instant query
GET /api/v1/query?query=<expr>

# Range query
GET /api/v1/query_range?query=<expr>&start=<time>&end=<time>&step=<duration>
```

### Targets

```bash
# List targets
GET /api/v1/targets
```

### Examples

```bash
# Get all up metrics
curl 'http://192.168.4.63:30090/api/v1/query?query=up'

# Get target status
curl -s http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'
```

## Loki API

Base URL: `http://192.168.4.63:31100`

### Health

```bash
# Ready check
GET /ready
```

### Query

```bash
# List labels
GET /loki/api/v1/labels

# Label values
GET /loki/api/v1/label/<name>/values

# Query logs
GET /loki/api/v1/query_range?query=<logql>&limit=<n>
```

### Examples

```bash
# Get labels
curl -s http://192.168.4.63:31100/loki/api/v1/labels | jq

# Query logs
curl -s 'http://192.168.4.63:31100/loki/api/v1/query_range?query={namespace="monitoring"}&limit=10'
```

## Grafana API

Base URL: `http://192.168.4.63:30300`

### Health

```bash
# Health check
GET /api/health
```

### Datasources

```bash
# List datasources
GET /api/datasources

# Get datasource by ID
GET /api/datasources/<id>
```

### Dashboards

```bash
# Search dashboards
GET /api/search

# Get dashboard by UID
GET /api/dashboards/uid/<uid>
```

### Examples

```bash
# Check health
curl http://192.168.4.63:30300/api/health

# List datasources
curl -s http://192.168.4.63:30300/api/datasources | jq '.[].name'

# List dashboards
curl -s http://192.168.4.63:30300/api/search | jq '.[].title'
```

## Kubernetes API

Access via kubectl or direct API calls.

### Configuration

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### Examples

```bash
# Get nodes
kubectl get nodes -o json

# Get pods
kubectl get pods -A -o json

# Direct API call
curl -k --cert /etc/kubernetes/pki/admin.crt \
  --key /etc/kubernetes/pki/admin.key \
  https://192.168.4.63:6443/api/v1/nodes
```

## Related Documentation

- [Prometheus](../components/monitoring/prometheus.md)
- [Grafana](../components/monitoring/grafana.md)
- [Loki](../components/monitoring/loki.md)
