# Common Issues

This guide covers frequently encountered issues and their solutions.

## Node Issues

### Node NotReady

**Symptoms:**
- `kubectl get nodes` shows NotReady
- Pods pending or evicted

**Diagnosis:**
```bash
kubectl describe node <node>
ssh <node> "systemctl status kubelet"
ssh <node> "journalctl -u kubelet -n 50"
```

**Common Causes:**
1. Kubelet not running
2. Network issues
3. Node sleeping (WoL)

**Solutions:**

```bash
# Wake sleeping node
wakeonlan <MAC>

# Restart kubelet
ssh <node> "systemctl restart kubelet"

# Check network
ssh <node> "ping 192.168.4.63"
```

### Node Auto-Sleeping

**Symptoms:**
- Node becomes NotReady unexpectedly
- SSH connection refused

**Solution:**
```bash
# Wake node
wakeonlan b8:ac:6f:7e:6c:9d  # storagenodet3500
wakeonlan d0:94:66:30:d6:63  # homelab

# Disable auto-sleep temporarily
ssh <node> "systemctl stop vmstation-autosleep.timer"
```

## Pod Issues

### CrashLoopBackOff

**Diagnosis:**
```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

**Common Causes:**
1. Configuration error
2. Missing dependencies
3. Permission issues
4. Resource limits

**Solutions by Cause:**

**Configuration:**
```bash
kubectl get configmap <config> -n <namespace> -o yaml
```

**Permissions:**
```bash
# Check SecurityContext
kubectl get pod <pod> -n <namespace> -o yaml | grep -A10 securityContext
```

### Pod Pending

**Diagnosis:**
```bash
kubectl describe pod <pod> -n <namespace>
```

**Common Causes:**
1. No schedulable nodes
2. PVC pending
3. Resource constraints
4. Node selector mismatch

**Solutions:**

```bash
# Check node taints
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

# Check PVCs
kubectl get pvc -n <namespace>

# Check resources
kubectl top nodes
```

### ImagePullBackOff

**Diagnosis:**
```bash
kubectl describe pod <pod> -n <namespace> | grep -A5 "Events"
```

**Solutions:**
```bash
# Check image name
kubectl get pod <pod> -n <namespace> -o jsonpath='{.spec.containers[*].image}'

# Test pull manually
ssh <node> "crictl pull <image>"
```

## Storage Issues

### PVC Pending

**Symptoms:**
- PVC shows Pending status
- Pods waiting for volume

**Diagnosis:**
```bash
kubectl get pvc -n <namespace>
kubectl describe pvc <pvc> -n <namespace>
kubectl get sc
```

**Solution:**
```bash
# Deploy storage class
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Permission Denied

**Symptoms:**
- Pod can't write to volume
- Permission denied in logs

**Solution:**
```bash
# Fix Prometheus
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus

# Fix Loki
sudo chown -R 10001:10001 /srv/monitoring_data/loki

# Fix Grafana
sudo chown -R 472:472 /srv/monitoring_data/grafana
```

## Network Issues

### DNS Resolution Failed

**Symptoms:**
- Pods can't resolve service names
- "Name or service not known" errors

**Diagnosis:**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl run test --rm -it --image=busybox -- nslookup kubernetes
```

**Solution:**
```bash
kubectl rollout restart deployment/coredns -n kube-system
```

### Service Not Accessible

**Symptoms:**
- Can't reach NodePort services
- Connection refused

**Diagnosis:**
```bash
kubectl get svc <service> -n <namespace>
kubectl get endpoints <service> -n <namespace>
```

**Solutions:**
```bash
# Check endpoints
kubectl describe endpoints <service> -n <namespace>

# Check firewall
ssh <node> "iptables -L -n | grep <port>"
```

## Deployment Issues

### SSH Key Errors

**Symptoms:**
- Ansible can't connect
- Permission denied (publickey)

**Solution:**
```yaml
# Use absolute paths in inventory
ansible_ssh_private_key_file: /root/.ssh/id_k3s  # Correct
# NOT: ~/.ssh/id_k3s
```

### Kubelet Path Wrong

**Symptoms:**
- Node won't join
- Kubelet not found

**Solution:**
```bash
# Create symlink
ssh <node> "ln -sf /usr/local/bin/kubelet /usr/bin/kubelet"
ssh <node> "systemctl restart kubelet"
```

### Version Mismatch

**Symptoms:**
- Node joins but shows different version
- Features not working

**Solution:**
```bash
# Upgrade kubelet
ssh <node> "apt install kubelet=1.29.0-1.1"
# Or use Kubespray upgrade
```

## Monitoring Issues

### Prometheus Down

**Quick Fix:**
```bash
kubectl delete pod prometheus-0 -n monitoring
```

**Check Logs:**
```bash
kubectl logs prometheus-0 -n monitoring -c prometheus
```

### Grafana Not Loading

**Check Status:**
```bash
kubectl get pod -n monitoring -l app=grafana
curl http://192.168.4.63:30300/api/health
```

**Restart:**
```bash
kubectl rollout restart deployment/grafana -n monitoring
```

### Loki Not Ready

**Check Logs:**
```bash
kubectl logs loki-0 -n monitoring
```

**Common Fix:**
```bash
sudo chown -R 10001:10001 /srv/monitoring_data/loki
kubectl delete pod loki-0 -n monitoring
```

## Getting More Help

### Collect Diagnostics

```bash
./scripts/diagnose-monitoring-stack.sh
```

### View Artifacts

```bash
ls -lh ansible/artifacts/
```

### Validation

```bash
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Related Documentation

- [Monitoring Issues](monitoring-issues.md)
- [Storage Issues](storage-issues.md)
- [Network Issues](network-issues.md)
- [Diagnostic Procedures](diagnostic-procedures.md)
