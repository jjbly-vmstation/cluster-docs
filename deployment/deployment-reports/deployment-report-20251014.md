# Deployment Report 2025-10-14

**Date:** 2025-10-14  
**Deployment Method:** Kubespray  
**Target:** 3-node cluster (1 control-plane, 2 workers)

## Summary

### Successful Components

✅ **Control Plane:** masternode (192.168.4.63) - Ready, v1.29.15  
✅ **Worker Node 1:** storagenodet3500 (192.168.4.61) - Ready  
✅ **Worker Node 2:** homelab (192.168.4.62) - Ready (v1.28.6)  
✅ **Monitoring Stack:** Prometheus, Grafana, Loki deployed  
✅ **Infrastructure:** NTP/Chrony running  
✅ **Network Plugin:** Calico CNI  

## Issues Encountered & Fixes

### 1. SSH Key Path Configuration

**Issue:** Ansible inventory used relative paths `~/.ssh/id_k3s`  
**Impact:** SSH authentication failures  
**Fix:** Changed to absolute paths `/root/.ssh/id_k3s`

```yaml
# Before
ansible_ssh_private_key_file: ~/.ssh/id_k3s

# After
ansible_ssh_private_key_file: /root/.ssh/id_k3s
```

### 2. homelab Kubelet Binary Path

**Issue:** Kubelet at `/usr/local/bin/kubelet`, systemd expected `/usr/bin/kubelet`  
**Symptoms:** Node failed to join, kubelet service errors  
**Root Cause:** Previous RKE2 installation  
**Fix:**

```bash
sudo ln -sf /usr/local/bin/kubelet /usr/bin/kubelet
sudo systemctl restart kubelet
```

### 3. storagenodet3500 Sleep Mode

**Issue:** Node auto-sleeping due to stale counters  
**Impact:** Node NotReady, pods pending  
**Fix:**

```bash
wakeonlan b8:ac:6f:7e:6c:9d
# Disable auto-sleep or reset counters
```

### 4. Storage Class Missing (Resolved)

**Issue:** No default storage class  
**Impact:** PVCs stuck in Pending  
**Fix:**

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### 5. Multus CNI Plugin (Minor)

**Issue:** Template error for Multus  
**Impact:** Secondary networks unavailable  
**Status:** Ignored - basic networking functional

## Current Cluster State

### Nodes

```
NAME               STATUS   ROLES           AGE   VERSION
masternode         Ready    control-plane   60m   v1.29.15
storagenodet3500   Ready    <none>          59m   v1.29.15
homelab            Ready    <none>          45m   v1.28.6
```

### Running Pods

**kube-system:** All control plane components healthy  
**monitoring:** All monitoring pods running  
**infrastructure:** Chrony NTP running  

## Access Information

### Service URLs

| Service | URL |
|---------|-----|
| Grafana | http://192.168.4.63:30300 |
| Prometheus | http://192.168.4.63:30090 |
| Loki | http://192.168.4.63:31100 |

### Kubeconfig

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## Verification Commands

```bash
# Check cluster status
kubectl get nodes -o wide
kubectl get pods -A

# Verify monitoring
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get pvc -n monitoring
```

## Files Modified

- /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml` - SSH key paths fixed

## Lessons Learned

1. **Use absolute paths** in Ansible inventory
2. **Check kubelet binary location** on RHEL nodes
3. **Reset sleep counters** before deployment
4. **Deploy storage class** before stateful workloads

## Related Documentation

- [Deployment Final Status](deployment-final-status.md)
- [Troubleshooting](../../troubleshooting/README.md)
- [Cluster Deployment](../cluster-deployment.md)
