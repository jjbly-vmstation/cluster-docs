# Common Issues

Frequently encountered problems and solutions.

## Node Issues

### Node NotReady

**Symptoms:**
- `kubectl get nodes` shows NotReady
- Pods stuck in Pending

**Causes and Fixes:**

1. **Kubelet not running**
   ```bash
   ssh <node> "systemctl status kubelet"
   ssh <node> "systemctl restart kubelet"
   ```

2. **Network issues**
   ```bash
   ssh <node> "ping -c 3 192.168.4.63"
   ```

3. **Node sleeping (auto-sleep)**
   ```bash
   wakeonlan <mac-address>
   ```

4. **Disk full**
   ```bash
   ssh <node> "df -h"
   ```

### Node NotSchedulable

**Symptoms:**
- Pods not scheduling on node
- Node shows `SchedulingDisabled`

**Fix:**
```bash
kubectl uncordon <nodename>
```

## Pod Issues

### Pod Pending

**Symptoms:**
- Pod stuck in Pending state

**Causes and Fixes:**

1. **No available nodes**
   ```bash
   kubectl describe pod <pod> | grep -A5 Events
   kubectl get nodes
   ```

2. **Insufficient resources**
   ```bash
   kubectl describe pod <pod> | grep -A5 Events
   kubectl top nodes
   ```

3. **PVC pending**
   ```bash
   kubectl get pvc -n <namespace>
   kubectl get sc
   ```

### Pod CrashLoopBackOff

**Symptoms:**
- Pod restarting repeatedly
- Status shows CrashLoopBackOff

**Diagnosis:**
```bash
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl describe pod <pod> -n <namespace>
```

**Common causes:**
- Application error
- Missing configuration
- Permission denied
- Resource limits exceeded

### Pod ImagePullBackOff

**Symptoms:**
- Pod stuck in ImagePullBackOff

**Causes and Fixes:**

1. **Wrong image name/tag**
   ```bash
   kubectl describe pod <pod> | grep Image
   ```

2. **Registry unreachable**
   ```bash
   kubectl describe pod <pod> | grep -A5 Events
   ```

3. **Missing image pull secret**
   ```bash
   kubectl get secrets -n <namespace>
   ```

## Service Issues

### Service No Endpoints

**Symptoms:**
- Service shows `<none>` for endpoints
- Cannot access service

**Causes and Fixes:**

1. **No matching pods**
   ```bash
   kubectl get endpoints <service> -n <namespace>
   kubectl get pods -l <selector-labels> -n <namespace>
   ```

2. **Pods not Ready**
   ```bash
   kubectl get pods -n <namespace>
   ```

3. **Wrong selector**
   ```bash
   kubectl describe svc <service> -n <namespace>
   kubectl describe pod <pod> -n <namespace> | grep Labels
   ```

### Cannot Access NodePort

**Symptoms:**
- `curl http://192.168.4.63:<nodeport>` fails

**Causes and Fixes:**

1. **Firewall blocking**
   ```bash
   ssh <node> "iptables -L -n | grep <port>"
   ```

2. **Service not running**
   ```bash
   kubectl get svc <service> -n <namespace>
   kubectl get endpoints <service> -n <namespace>
   ```

## Storage Issues

### PVC Pending

See [Storage Issues](storage-issues.md)

### Permission Denied

```bash
# Check ownership
ls -la /srv/monitoring_data/pvc-*/

# Fix Prometheus
sudo chown -R 65534:65534 /srv/monitoring_data/*prometheus*

# Fix Loki
sudo chown -R 10001:10001 /srv/monitoring_data/*loki*

# Fix Grafana
sudo chown -R 472:472 /srv/monitoring_data/*grafana*
```

## Network Issues

### DNS Not Resolving

See [Network Issues](network-issues.md)

### Pod Cannot Reach External

```bash
# Test from pod
kubectl exec -it <pod> -- ping 8.8.8.8
kubectl exec -it <pod> -- nslookup google.com

# Check DNS config
kubectl get configmap coredns -n kube-system -o yaml
```

## Monitoring Issues

See [Monitoring Issues](monitoring-issues.md)

## Quick Recovery

### Reset and Redeploy

```bash
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray  # or debian
./scripts/validate-monitoring-stack.sh
```

### Restart Monitoring Stack

```bash
kubectl rollout restart statefulset prometheus -n monitoring
kubectl rollout restart statefulset loki -n monitoring
kubectl rollout restart deployment grafana -n monitoring
```

## Related Documentation

- [Monitoring Issues](monitoring-issues.md)
- [Storage Issues](storage-issues.md)
- [Network Issues](network-issues.md)
- [Diagnostic Procedures](diagnostic-procedures.md)
