# Cluster Management

Managing Kubernetes cluster resources.

## Node Operations

### View Nodes

```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <nodename>
```

### Node Labels

```bash
# Add label
kubectl label node <nodename> role=storage

# Remove label
kubectl label node <nodename> role-

# View labels
kubectl get nodes --show-labels
```

### Node Taints

```bash
# Add taint
kubectl taint node <nodename> key=value:NoSchedule

# Remove taint
kubectl taint node <nodename> key-

# View taints
kubectl describe node <nodename> | grep Taints
```

## Workload Scheduling

### NodeSelector

Schedule on specific node:

```yaml
spec:
  nodeSelector:
    kubernetes.io/hostname: storagenodet3500
```

### Tolerations

Allow scheduling on tainted nodes:

```yaml
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

### Affinity

Advanced scheduling:

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: role
            operator: In
            values:
            - storage
```

## Deployment Management

### Scale Deployment

```bash
kubectl scale deployment <name> --replicas=3 -n <namespace>
```

### Rolling Restart

```bash
kubectl rollout restart deployment <name> -n <namespace>
```

### Check Rollout Status

```bash
kubectl rollout status deployment <name> -n <namespace>
```

### Rollback

```bash
kubectl rollout undo deployment <name> -n <namespace>
kubectl rollout undo deployment <name> --to-revision=2 -n <namespace>
```

## StatefulSet Management

### Scale StatefulSet

```bash
kubectl scale statefulset <name> --replicas=1 -n <namespace>
```

### Restart StatefulSet

```bash
kubectl rollout restart statefulset <name> -n <namespace>
```

### Delete Pod to Restart

```bash
kubectl delete pod <name>-0 -n <namespace>
```

## DaemonSet Management

### View DaemonSets

```bash
kubectl get daemonset -A
```

### Rolling Update

DaemonSets update automatically when configuration changes.

## Namespace Management

### List Namespaces

```bash
kubectl get namespaces
```

### Create Namespace

```bash
kubectl create namespace myapp
```

### Delete Namespace

```bash
kubectl delete namespace myapp
# Warning: Deletes all resources in namespace
```

## Resource Limits

### View Resource Usage

```bash
kubectl top nodes
kubectl top pods -A
```

### Set Resource Limits

```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

## ConfigMap and Secrets

### View ConfigMaps

```bash
kubectl get configmap -n <namespace>
kubectl describe configmap <name> -n <namespace>
```

### Create ConfigMap

```bash
kubectl create configmap myconfig --from-file=config.yaml -n <namespace>
```

### View Secrets

```bash
kubectl get secrets -n <namespace>
kubectl describe secret <name> -n <namespace>
```

### Decode Secret

```bash
kubectl get secret <name> -n <namespace> -o jsonpath='{.data.password}' | base64 -d
```

## Related Documentation

- [Day-2 Operations](day-2-operations.md)
- [Scaling](scaling.md)
- [Upgrades](upgrades.md)
