# Cluster Management

This guide covers Kubernetes cluster management tasks.

## Cluster Access

### Set KUBECONFIG

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf

# Or add to profile
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bashrc
```

### Verify Access

```bash
kubectl cluster-info
kubectl get nodes
```

## Node Management

### View Nodes

```bash
kubectl get nodes -o wide
```

### Node Details

```bash
kubectl describe node <node-name>
```

### Cordon Node

Prevent new pods from scheduling:

```bash
kubectl cordon <node>
```

### Drain Node

Evict pods for maintenance:

```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
```

### Uncordon Node

Return node to service:

```bash
kubectl uncordon <node>
```

### Label Nodes

```bash
# Add label
kubectl label node <node> role=storage

# Remove label
kubectl label node <node> role-
```

### Taint Nodes

```bash
# Add taint
kubectl taint nodes <node> key=value:NoSchedule

# Remove taint
kubectl taint nodes <node> key=value:NoSchedule-
```

## Namespace Management

### Create Namespace

```bash
kubectl create namespace <name>
```

### List Namespaces

```bash
kubectl get namespaces
```

### Delete Namespace

```bash
kubectl delete namespace <name>
```

### Set Default Namespace

```bash
kubectl config set-context --current --namespace=<namespace>
```

## Workload Management

### Deployments

```bash
# List deployments
kubectl get deployments -n <namespace>

# Scale
kubectl scale deployment <name> --replicas=3 -n <namespace>

# Rollout status
kubectl rollout status deployment/<name> -n <namespace>

# Rollback
kubectl rollout undo deployment/<name> -n <namespace>
```

### StatefulSets

```bash
# List statefulsets
kubectl get statefulsets -n <namespace>

# Scale
kubectl scale statefulset <name> --replicas=3 -n <namespace>

# Delete pod (will recreate)
kubectl delete pod <statefulset-pod-0> -n <namespace>
```

### DaemonSets

```bash
# List daemonsets
kubectl get daemonsets -A

# View details
kubectl describe daemonset <name> -n <namespace>
```

## Resource Management

### View Resources

```bash
# Node resources
kubectl top nodes

# Pod resources
kubectl top pods -n <namespace>
```

### Resource Quotas

```bash
# View quotas
kubectl get resourcequotas -n <namespace>

# Create quota
kubectl create resourcequota <name> --hard=cpu=4,memory=8Gi -n <namespace>
```

### Limit Ranges

```bash
kubectl get limitranges -n <namespace>
```

## Configuration Management

### ConfigMaps

```bash
# Create from file
kubectl create configmap <name> --from-file=<path> -n <namespace>

# View
kubectl get configmap <name> -o yaml -n <namespace>

# Edit
kubectl edit configmap <name> -n <namespace>
```

### Secrets

```bash
# Create
kubectl create secret generic <name> --from-literal=key=value -n <namespace>

# View (decoded)
kubectl get secret <name> -o jsonpath='{.data.key}' -n <namespace> | base64 -d
```

## Service Management

### View Services

```bash
kubectl get svc -n <namespace>
```

### Expose Deployment

```bash
kubectl expose deployment <name> --port=80 --target-port=8080 --type=NodePort -n <namespace>
```

### Port Forward

```bash
kubectl port-forward svc/<service> 8080:80 -n <namespace>
```

## Troubleshooting Commands

### Pod Issues

```bash
# Events
kubectl get events --sort-by='.lastTimestamp' -n <namespace>

# Describe
kubectl describe pod <pod> -n <namespace>

# Logs
kubectl logs <pod> -n <namespace>

# Previous logs
kubectl logs <pod> --previous -n <namespace>

# Exec into pod
kubectl exec -it <pod> -n <namespace> -- /bin/sh
```

### Network Issues

```bash
# Test DNS
kubectl run test --rm -it --image=busybox -- nslookup <service>.<namespace>

# Test connectivity
kubectl run test --rm -it --image=busybox -- wget -O- <service>.<namespace>:<port>
```

## Cluster Maintenance

### Backup etcd

```bash
# On control plane
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### View Cluster Info

```bash
kubectl cluster-info
kubectl cluster-info dump
```

### Check Component Status

```bash
kubectl get componentstatuses
```

## Related Documentation

- [Day-2 Operations](day-2-operations.md)
- [Upgrades](upgrades.md)
- [Troubleshooting](../troubleshooting/common-issues.md)
