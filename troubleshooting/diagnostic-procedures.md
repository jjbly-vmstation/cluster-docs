# Diagnostic Procedures

Systematic debugging approaches.

## Diagnostic Tools

### Automated Diagnostics

```bash
# Comprehensive monitoring diagnostics
./scripts/diagnose-monitoring-stack.sh

# Creates timestamped directory with:
# - Pod status and logs
# - Service and endpoint configs
# - PVC/PV status
# - ConfigMaps
# - Host directory permissions
# - Analysis and recommendations
```

### Validation Scripts

```bash
# Validate monitoring stack
./scripts/validate-monitoring-stack.sh

# Complete validation suite
./tests/test-complete-validation.sh
```

### Remediation

```bash
# Automated fix for common issues
./scripts/remediate-monitoring-stack.sh
```

## Systematic Debugging

### Step 1: Check Nodes

```bash
kubectl get nodes -o wide
```

**Questions:**
- All nodes Ready?
- Correct versions?
- Correct IPs?

### Step 2: Check Pods

```bash
kubectl get pods -A | grep -v Running
```

**For problematic pods:**
```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

### Step 3: Check Events

```bash
kubectl get events --sort-by='.lastTimestamp' -A | tail -30
```

### Step 4: Check Services

```bash
kubectl get svc -A
kubectl get endpoints -A
```

### Step 5: Check Resources

```bash
kubectl top nodes
kubectl top pods -A
```

## Component-Specific Diagnostics

### Kubernetes API

```bash
kubectl cluster-info
kubectl get componentstatuses
```

### etcd

```bash
kubectl -n kube-system get pods -l component=etcd
kubectl -n kube-system logs <etcd-pod>
```

### kubelet

```bash
ssh <node> "systemctl status kubelet"
ssh <node> "journalctl -xeu kubelet -n 50"
```

### CoreDNS

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### CNI

```bash
# Calico
kubectl get pods -n kube-system -l k8s-app=calico-node
kubectl logs -n kube-system -l k8s-app=calico-node

# Flannel
kubectl get pods -n kube-flannel
kubectl logs -n kube-flannel <pod>
```

## Log Analysis

### Filter Error Logs

```bash
kubectl logs <pod> -n <namespace> | grep -i error
kubectl logs <pod> -n <namespace> | grep -i fail
kubectl logs <pod> -n <namespace> | grep -i panic
```

### Follow Logs

```bash
kubectl logs -f <pod> -n <namespace>
```

### Multi-container Pods

```bash
kubectl logs <pod> -n <namespace> -c <container>
```

### Previous Instance

```bash
kubectl logs <pod> -n <namespace> --previous
```

## Resource Debugging

### Describe Resource

```bash
kubectl describe <resource> <name> -n <namespace>
```

### Get YAML

```bash
kubectl get <resource> <name> -n <namespace> -o yaml
```

### Get JSON Path

```bash
kubectl get pod <pod> -n <namespace> -o jsonpath='{.status.phase}'
```

## Network Debugging

### DNS Test

```bash
kubectl run test --rm -it --image=busybox -- nslookup kubernetes.default
```

### Connectivity Test

```bash
kubectl run test --rm -it --image=busybox -- wget -O- http://<service>:<port>
```

### Full Network Debug

```bash
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash
```

## Storage Debugging

### Check PVC

```bash
kubectl get pvc -A
kubectl describe pvc <name> -n <namespace>
```

### Check PV

```bash
kubectl get pv
kubectl describe pv <name>
```

### Check Storage Class

```bash
kubectl get sc
kubectl describe sc <name>
```

### Check Provisioner

```bash
kubectl get pods -n local-path-storage
kubectl logs -n local-path-storage <pod>
```

## Creating Debug Reports

### Collect All Information

```bash
#!/bin/bash
REPORT_DIR=/tmp/k8s-debug-$(date +%Y%m%d-%H%M%S)
mkdir -p $REPORT_DIR

# Cluster info
kubectl cluster-info > $REPORT_DIR/cluster-info.txt
kubectl get nodes -o wide > $REPORT_DIR/nodes.txt
kubectl get pods -A > $REPORT_DIR/pods.txt
kubectl get events -A --sort-by='.lastTimestamp' > $REPORT_DIR/events.txt
kubectl get svc -A > $REPORT_DIR/services.txt
kubectl get pvc -A > $REPORT_DIR/pvcs.txt

# Specific namespace
NAMESPACE=monitoring
kubectl describe pods -n $NAMESPACE > $REPORT_DIR/describe-pods.txt
kubectl logs -n $NAMESPACE --all-containers --prefix > $REPORT_DIR/logs.txt 2>&1

echo "Report saved to $REPORT_DIR"
```

## Escalation Path

1. **Self-service**
   - Check documentation
   - Run diagnostic scripts
   - Review logs

2. **Automated remediation**
   - Run remediation scripts
   - Restart affected components

3. **Manual intervention**
   - Apply targeted fixes
   - Reset and redeploy if needed

## Related Documentation

- [Common Issues](common-issues.md)
- [Monitoring Issues](monitoring-issues.md)
- [Network Issues](network-issues.md)
- [Storage Issues](storage-issues.md)
