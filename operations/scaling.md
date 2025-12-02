# Scaling

Adding and removing cluster capacity.

## Horizontal Scaling

### Scale Deployments

```bash
# Scale up
kubectl scale deployment <name> --replicas=3 -n <namespace>

# Scale down
kubectl scale deployment <name> --replicas=1 -n <namespace>
```

### Scale StatefulSets

```bash
kubectl scale statefulset <name> --replicas=2 -n <namespace>
```

### Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

## Adding Worker Nodes

### Step 1: Prepare Node

```bash
# Install prerequisites
apt update
apt install -y containerd apt-transport-https ca-certificates curl

# Configure networking
modprobe br_netfilter
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 > /proc/sys/net/ipv4/ip_forward

# Disable swap
swapoff -a
```

### Step 2: Add to Inventory

Edit /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml`:

```yaml
storage_nodes:
  hosts:
    storagenodet3500:
      ansible_host: 192.168.4.61
      # ...
    newnode:
      ansible_host: 192.168.4.64
      ansible_user: root
      ansible_ssh_private_key_file: /root/.ssh/id_k3s
      mac_address: "aa:bb:cc:dd:ee:ff"
```

### Step 3: Run Deployment

```bash
./deploy.sh debian
```

### Step 4: Verify

```bash
kubectl get nodes
```

## Removing Worker Nodes

### Step 1: Drain Node

```bash
kubectl drain <nodename> --ignore-daemonsets --delete-emptydir-data
```

### Step 2: Delete Node

```bash
kubectl delete node <nodename>
```

### Step 3: Reset Node

```bash
# On the node
kubeadm reset -f
rm -rf /etc/cni/net.d /var/lib/kubelet /etc/kubernetes
```

### Step 4: Update Inventory

Remove node from /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml`.

## Resource Scaling

### Increase Pod Resources

```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "512Mi"  # Increased
        cpu: "500m"
      limits:
        memory: "1Gi"    # Increased
        cpu: "1000m"
```

### Vertical Pod Autoscaler (VPA)

Currently not deployed. For future consideration:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: Auto
```

## Storage Scaling

### Expand PVC

```bash
# Check if storage class supports expansion
kubectl get sc local-path -o yaml | grep allowVolumeExpansion

# Edit PVC
kubectl edit pvc <pvc-name> -n <namespace>
# Change spec.resources.requests.storage
```

### Add Storage Node

Currently storage is on masternode only. To add storage nodes:

1. Configure local-path-provisioner with additional nodes
2. Update node path map in ConfigMap

## Monitoring Scaling

### Prometheus Retention

```yaml
args:
  - '--storage.tsdb.retention.time=30d'  # Increase retention
  - '--storage.tsdb.retention.size=16GB' # Increase size
```

### Loki Retention

```yaml
limits_config:
  retention_period: 1488h  # 62 days
```

## Capacity Planning

### Current Capacity

| Node | CPU | RAM | Disk |
|------|-----|-----|------|
| masternode | 4 | 8GB | 100GB |
| storagenodet3500 | 4 | 8GB | 500GB |
| homelab | 4 | 8GB | 100GB |

### Utilization Queries

```promql
# CPU utilization
100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory utilization
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Disk utilization
(1 - node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100
```

## Related Documentation

- [Cluster Management](cluster-management.md)
- [Architecture Overview](../architecture/overview.md)
- [Upgrades](upgrades.md)
