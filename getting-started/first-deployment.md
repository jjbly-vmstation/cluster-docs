# First Deployment

After installing VMStation, this guide helps you verify your cluster and deploy your first application.

## Verify Cluster Health

### Check Nodes

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes -o wide
```

Expected output:
```
NAME               STATUS   ROLES           AGE   VERSION    INTERNAL-IP    OS-IMAGE       
masternode         Ready    control-plane   10m   v1.29.15   192.168.4.63   Debian 12
storagenodet3500   Ready    <none>          9m    v1.29.15   192.168.4.61   Debian 12
homelab            Ready    <none>          8m    v1.29.15   192.168.4.62   RHEL 10
```

### Check System Pods

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods -n kube-system
```

All pods should be `Running` or `Completed`.

### Check Monitoring Stack

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods -n monitoring
```

Expected components:
- prometheus-0
- grafana-*
- loki-0
- promtail-*
- node-exporter-*

## Access Monitoring Services

### Grafana Dashboard

1. Open http://192.168.4.63:30300
2. Login: admin / admin (change on first login)
3. Navigate to Dashboards
4. Explore pre-configured dashboards

### Prometheus Metrics

1. Open http://192.168.4.63:30090
2. Go to Status → Targets
3. Verify all targets are UP

### Query Logs with Loki

1. Open Grafana
2. Go to Explore
3. Select Loki datasource
4. Query: `{namespace="monitoring"}`

## Deploy a Test Application

### Create a Namespace

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf create namespace test-app
```

### Deploy nginx

```yaml
# Save as test-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  namespace: test-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-test
  template:
    metadata:
      labels:
        app: nginx-test
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
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
    app: nginx-test
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

Apply the configuration:

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf apply -f test-nginx.yaml
```

### Verify Deployment

```bash
# Check pods
kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods -n test-app

# Check service
kubectl --kubeconfig=/etc/kubernetes/admin.conf get svc -n test-app

# Test access
curl http://192.168.4.63:30080
```

### View Logs in Grafana

1. Open Grafana → Explore
2. Select Loki datasource
3. Query: `{namespace="test-app"}`

## Deploy Jellyfin (Media Server)

VMStation includes Jellyfin for media streaming.

### Apply Jellyfin Manifest

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf apply -f manifests/apps/jellyfin.yaml
```

### Access Jellyfin

- URL: http://192.168.4.63:30096
- Complete the setup wizard

### Configure Media Storage

Jellyfin uses hostPath volumes. Configure media directories on storagenodet3500:

```bash
# On storagenodet3500
sudo mkdir -p /media/{movies,tv,music}
sudo chown -R 1000:1000 /media
```

## Run Validation Tests

### Complete Validation Suite

```bash
./tests/test-complete-validation.sh
```

This tests:
- Node status
- Pod health
- Monitoring connectivity
- Service accessibility

### Individual Tests

```bash
# Monitoring health
./tests/test-monitoring-exporters-health.sh

# Monitoring access
./tests/test-monitoring-access.sh

# Loki validation
./tests/test-loki-validation.sh
```

## Clean Up Test Application

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf delete namespace test-app
```

## Next Steps

### Operations
- [Day-2 Operations](../operations/day-2-operations.md) - Ongoing management
- [Power Management](../operations/power-management.md) - Auto-sleep configuration

### Components
- [Prometheus](../components/monitoring/prometheus.md) - Configure metrics
- [Grafana](../components/monitoring/grafana.md) - Customize dashboards

### Troubleshooting
- [Common Issues](../troubleshooting/common-issues.md) - If something goes wrong

## Tips

### Set KUBECONFIG

Add to your shell profile:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### Create Aliases

```bash
alias k='kubectl --kubeconfig=/etc/kubernetes/admin.conf'
alias kgp='kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods'
alias kga='kubectl --kubeconfig=/etc/kubernetes/admin.conf get all'
```

### Enable Tab Completion

```bash
# Bash
source <(kubectl completion bash)

# Zsh
source <(kubectl completion zsh)
```
