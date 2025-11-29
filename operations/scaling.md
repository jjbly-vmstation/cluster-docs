# Scaling

This guide covers scaling VMStation infrastructure.

## Scaling Overview

VMStation supports scaling:
- **Horizontal**: Add/remove nodes
- **Vertical**: Increase node resources
- **Workload**: Scale applications

## Add Worker Node

### Prerequisites

- Node meets hardware requirements
- Network connectivity to cluster
- SSH access configured

### Step 1: Update Inventory

Edit `ansible/inventory/hosts.yml`:

```yaml
storage_nodes:
  hosts:
    storagenodet3500:
      ansible_host: 192.168.4.61
    newnode:
      ansible_host: 192.168.4.64
      ansible_user: root
      ansible_ssh_private_key_file: /root/.ssh/id_k3s
```

### Step 2: Add to Kubespray Inventory

Edit `.cache/kubespray/inventory/mycluster/inventory.ini`:

```ini
[kube_node]
storagenodet3500
homelab
newnode
```

### Step 3: Run Scale Playbook

```bash
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini scale.yml -b
```

### Step 4: Verify Node

```bash
kubectl get nodes
# Should show newnode as Ready
```

## Remove Worker Node

### Step 1: Drain Node

```bash
kubectl drain newnode --ignore-daemonsets --delete-emptydir-data
```

### Step 2: Delete Node

```bash
kubectl delete node newnode
```

### Step 3: Clean Up Node

SSH to the node:

```bash
kubeadm reset -f
rm -rf /etc/cni/net.d
rm -rf /var/lib/kubelet
rm -rf /etc/kubernetes
```

### Step 4: Update Inventories

Remove node from both inventories.

## Scale Applications

### Scale Deployment

```bash
# Scale up
kubectl scale deployment <name> --replicas=3 -n <namespace>

# Scale down
kubectl scale deployment <name> --replicas=1 -n <namespace>
```

### Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

Apply:

```bash
kubectl apply -f hpa.yaml
```

### Vertical Pod Autoscaler (Optional)

For automatic resource adjustment:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
```

## Scale Monitoring Stack

### Increase Prometheus Storage

Edit `manifests/monitoring/prometheus.yaml`:

```yaml
volumeClaimTemplates:
- metadata:
    name: prometheus-storage
  spec:
    resources:
      requests:
        storage: 20Gi  # Increased from 10Gi
```

### Increase Loki Storage

Edit `manifests/monitoring/loki.yaml`:

```yaml
volumeClaimTemplates:
- metadata:
    name: loki-data
  spec:
    resources:
      requests:
        storage: 50Gi  # Increased from 20Gi
```

**Note**: Expanding PVCs may require recreating the StatefulSet.

### Add Prometheus Replicas (HA)

For high availability:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: prometheus
spec:
  replicas: 2  # Increased from 1
```

## Scale Storage

### Add Storage to Node

1. Attach additional disk to node
2. Format and mount:
   ```bash
   sudo mkfs.ext4 /dev/sdb
   sudo mkdir /mnt/additional
   sudo mount /dev/sdb /mnt/additional
   ```
3. Update local-path-provisioner config

### Move Storage Path

Edit local-path-provisioner config:

```yaml
data:
  config.json: |
    {
      "nodePathMap":[
        {
          "node":"DEFAULT_PATH_FOR_NON_LISTED_NODES",
          "paths":["/srv/monitoring_data"]
        },
        {
          "node":"storagenodet3500",
          "paths":["/mnt/additional"]
        }
      ]
    }
```

## Capacity Planning

### Current Resources

| Node | CPU | RAM | Disk |
|------|-----|-----|------|
| masternode | 4 | 8GB | 100GB |
| storagenodet3500 | 4 | 8GB | 500GB |
| homelab | 4 | 8GB | 100GB |

### Adding Capacity

Consider adding nodes when:
- CPU utilization > 70% sustained
- Memory utilization > 80%
- Disk usage > 80%

### Resource Monitoring

```bash
# Check node resources
kubectl top nodes

# Check pod resources
kubectl top pods -A
```

## Related Documentation

- [Cluster Management](cluster-management.md)
- [Architecture Overview](../architecture/overview.md)
- [Prerequisites](../getting-started/prerequisites.md)
