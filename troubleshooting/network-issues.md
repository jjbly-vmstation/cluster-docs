# Network Issues

This guide covers network-related troubleshooting.

## DNS Issues

### Service Name Resolution Failed

**Symptoms:**
```
dial tcp: lookup prometheus.monitoring.svc.cluster.local: no such host
```

**Diagnosis:**
```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Test DNS resolution
kubectl run test --rm -it --image=busybox -- nslookup kubernetes
```

**Solutions:**

**Restart CoreDNS:**
```bash
kubectl rollout restart deployment/coredns -n kube-system
```

**Check kubelet DNS config:**
```bash
ssh <node> "cat /var/lib/kubelet/config.yaml | grep clusterDNS"
# Should show: 10.233.0.3 (CoreDNS IP)
```

**Fix kubelet DNS:**
```bash
ssh <node> "sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml"
ssh <node> "systemctl restart kubelet"
```

### nodelocaldns Conflicts

**Symptoms:**
- Intermittent DNS failures
- nodelocaldns and CoreDNS both running

**Solution:**
```bash
# Remove nodelocaldns if using CoreDNS
kubectl delete daemonset nodelocaldns -n kube-system
```

## Service Connectivity

### Service Not Accessible

**Diagnosis:**
```bash
# Check service
kubectl get svc <service> -n <namespace>

# Check endpoints
kubectl get endpoints <service> -n <namespace>

# Test from another pod
kubectl run test --rm -it --image=busybox -- wget -O- <service>.<namespace>:<port>
```

**Common Issues:**
1. No endpoints (pods not Ready)
2. Wrong selector
3. Port mismatch

**Solutions:**

**Check selector:**
```bash
kubectl get svc <service> -n <namespace> -o yaml | grep -A5 selector
kubectl get pods -n <namespace> -l <selector>
```

**Check ports:**
```bash
kubectl get svc <service> -n <namespace> -o yaml | grep -A10 ports
```

### NodePort Not Working

**Diagnosis:**
```bash
# Test from node
curl http://localhost:<nodePort>

# Test from external
curl http://192.168.4.63:<nodePort>
```

**Check firewall:**
```bash
ssh <node> "iptables -L -n | grep <nodePort>"
```

**Open port:**
```bash
# Debian
ssh <node> "iptables -A INPUT -p tcp --dport <nodePort> -j ACCEPT"

# RHEL
ssh <node> "firewall-cmd --add-port=<nodePort>/tcp --permanent && firewall-cmd --reload"
```

## CNI Issues

### Calico Problems

**Diagnosis:**
```bash
kubectl get pods -n kube-system -l k8s-app=calico-node
kubectl logs -n kube-system -l k8s-app=calico-node
```

**Common Issues:**
1. BGP not establishing
2. IP pool exhausted
3. Policy blocking traffic

**Solutions:**

**Restart Calico:**
```bash
kubectl rollout restart daemonset/calico-node -n kube-system
```

**Check IP pools:**
```bash
kubectl get ippool -o yaml
```

### Pod-to-Pod Communication Failed

**Diagnosis:**
```bash
# Get pod IPs
kubectl get pods -o wide

# Test connectivity
kubectl exec <pod-a> -- ping <pod-b-ip>
```

**Check CNI:**
```bash
ssh <node> "ls /etc/cni/net.d/"
ssh <node> "cat /etc/cni/net.d/10-calico.conflist"
```

## Firewall Issues

### Kubernetes Ports Blocked

**Required Ports:**

| Port | Service |
|------|---------|
| 6443 | API Server |
| 10250 | Kubelet |
| 2379-2380 | etcd |
| 30000-32767 | NodePort |

**Check firewall:**
```bash
# Debian
ssh <node> "iptables -L INPUT -n"

# RHEL
ssh <node> "firewall-cmd --list-all"
```

**Open ports:**
```bash
# RHEL
ssh <node> "firewall-cmd --permanent --add-port=6443/tcp"
ssh <node> "firewall-cmd --permanent --add-port=10250/tcp"
ssh <node> "firewall-cmd --permanent --add-port=30000-32767/tcp"
ssh <node> "firewall-cmd --reload"
```

## Network Policy Issues

### Traffic Blocked by Policy

**Diagnosis:**
```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy> -n <namespace>
```

**Debug:**
```bash
# Temporarily delete policy
kubectl delete networkpolicy <policy> -n <namespace>
# Test connectivity
# Recreate policy if it was the issue
```

## External Access Issues

### Can't Reach Services from Outside

**Diagnosis:**
```bash
# From external machine
curl http://192.168.4.63:30300

# Check routing
traceroute 192.168.4.63
```

**Common Issues:**
1. Firewall on node
2. Firewall on router
3. Routing issues

**Solutions:**

**Check node firewall:**
```bash
ssh 192.168.4.63 "iptables -L INPUT -n | grep 30300"
```

**Check service:**
```bash
kubectl get svc -n monitoring | grep 30300
```

## Wake-on-LAN Network Issues

### Magic Packet Not Reaching Node

**Diagnosis:**
```bash
# Check from same network
wakeonlan -i 192.168.4.255 <MAC>

# Verify MAC
arp -a | grep 192.168.4.61
```

**Requirements:**
- Same broadcast domain
- Switch forwards broadcasts
- Node has WoL enabled in BIOS

## Related Documentation

- [Network Architecture](../architecture/network-architecture.md)
- [Common Issues](common-issues.md)
- [DNS Infrastructure](../components/infrastructure/dns.md)
