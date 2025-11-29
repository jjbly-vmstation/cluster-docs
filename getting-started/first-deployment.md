# First Deployment

Deploy your first workload on VMStation.

## Verify Cluster Status

Before deploying, ensure the cluster is healthy:

```bash
# Check nodes
kubectl get nodes
# Expected: All nodes in Ready state

# Check system pods
kubectl get pods -n kube-system
# Expected: All pods Running

# Check monitoring
kubectl get pods -n monitoring
# Expected: All pods Running
```

## Deploy a Test Application

### Create Namespace

```bash
kubectl create namespace test-app
```

### Deploy nginx

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  namespace: test-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-test
  namespace: test-app
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
EOF
```

### Verify Deployment

```bash
# Check pods
kubectl get pods -n test-app

# Check service
kubectl get svc -n test-app

# Test access
curl http://192.168.4.63:30080
```

### Clean Up

```bash
kubectl delete namespace test-app
```

## Access Monitoring

### Grafana

1. Open http://192.168.4.63:30300
2. Default login: admin/admin (change on first login)
3. Explore pre-configured dashboards

### Prometheus

1. Open http://192.168.4.63:30090
2. Query metrics
3. Check targets status

### Loki (via Grafana)

1. Open Grafana
2. Go to Explore
3. Select Loki datasource
4. Query logs: `{namespace="test-app"}`

## Deploy Jellyfin

Jellyfin media server is deployed automatically on storagenodet3500:

```bash
kubectl get pods -l app=jellyfin
```

Access at: http://192.168.4.63:30096

## Configure Workload Scheduling

### Schedule on Specific Node

```yaml
spec:
  nodeSelector:
    kubernetes.io/hostname: storagenodet3500
```

### Tolerate Auto-Sleep Nodes

For workloads on auto-sleep nodes:

```yaml
spec:
  tolerations:
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoSchedule"
```

## Next Steps

- [Day-2 Operations](../operations/day-2-operations.md) - Ongoing management
- [Monitoring](../components/monitoring/README.md) - Configure monitoring
- [Troubleshooting](../troubleshooting/README.md) - Common issues

## Related Documentation

- [Architecture Overview](../architecture/overview.md)
- [Cluster Deployment](../deployment/cluster-deployment.md)
- [Application Deployment](../deployment/application-deployment.md)
