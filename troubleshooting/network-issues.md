# Network Issues

Troubleshooting networking and DNS problems.

## DNS Issues

### DNS Not Resolving

**Symptoms:**
- Services can't find each other
- Error: "no such host"

**Check CoreDNS:**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

**Test from pod:**
```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
kubectl exec -it <pod> -- nslookup prometheus.monitoring.svc.cluster.local
```

### nodelocaldns Conflict

**Symptoms:**
- Intermittent DNS failures
- Grafana can't reach Prometheus

**Fix:**
```bash
# Remove nodelocaldns
kubectl delete daemonset nodelocaldns -n kube-system

# Update kubelet on all nodes
for node in masternode storagenodet3500 homelab; do
  ssh $node "sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml"
  ssh $node "systemctl restart kubelet"
done
```

### Redeploy CoreDNS

```bash
cd .cache/kubespray
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --tags coredns
```

### Check DNS Configuration

```bash
# CoreDNS service IP
kubectl get svc -n kube-system kube-dns

# Should be 10.233.0.3 or 10.96.0.10
```

## Pod Connectivity

### Pod Cannot Reach Other Pods

**Test connectivity:**
```bash
kubectl exec -it <pod1> -- ping <pod2-ip>
kubectl exec -it <pod1> -- curl <pod2-ip>:<port>
```

**Check CNI:**
```bash
kubectl get pods -n kube-system -l k8s-app=calico-node
# or
kubectl get pods -n kube-flannel
```

### Pod Cannot Reach External

**Test:**
```bash
kubectl exec -it <pod> -- ping 8.8.8.8
kubectl exec -it <pod> -- curl https://google.com
```

**Check NAT:**
```bash
iptables -t nat -L -n | grep MASQUERADE
```

## Service Issues

### Service Not Accessible

**Check service:**
```bash
kubectl get svc <name> -n <namespace>
kubectl get endpoints <name> -n <namespace>
```

**Test from cluster:**
```bash
kubectl run test --rm -it --image=busybox -- wget -O- http://<service>:<port>
```

### NodePort Not Working

**Check:**
```bash
kubectl get svc <name> -n <namespace>
# Note the NodePort

curl http://<node-ip>:<nodeport>
```

**Firewall:**
```bash
iptables -L -n | grep <nodeport>
```

## CNI Issues

### Flannel Issues

**Check pods:**
```bash
kubectl get pods -n kube-flannel
kubectl logs -n kube-flannel <flannel-pod>
```

**Check config:**
```bash
cat /etc/cni/net.d/10-flannel.conflist
```

### Calico Issues

**Check pods:**
```bash
kubectl get pods -n kube-system -l k8s-app=calico-node
kubectl logs -n kube-system -l k8s-app=calico-node
```

**Check status:**
```bash
calicoctl node status
```

### CNI Not Installed

**Symptoms:**
- Pods stuck in ContainerCreating
- Error: "network plugin is not ready"

**Fix:**
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

## Firewall Issues

### Check iptables

```bash
iptables -L -n
iptables -t nat -L -n
```

### Reset iptables

```bash
iptables -F
iptables -t nat -F
systemctl restart kubelet
```

### Required Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 6443 | TCP | Kubernetes API |
| 10250 | TCP | Kubelet |
| 30000-32767 | TCP | NodePort |

## Network Debugging

### Debug Pod

```bash
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash

# Inside pod
ping <ip>
curl <url>
nslookup <hostname>
traceroute <ip>
```

### Check Routes

```bash
# On node
ip route
```

### Check Interfaces

```bash
# On node
ip addr
```

## Recovery

### Restart kube-proxy

```bash
kubectl rollout restart daemonset kube-proxy -n kube-system
```

### Restart CNI

```bash
kubectl rollout restart daemonset calico-node -n kube-system
# or
kubectl rollout restart daemonset kube-flannel-ds -n kube-flannel
```

### Full Network Reset

```bash
# On each node
systemctl stop kubelet
rm -rf /etc/cni/net.d/*
ip link delete cni0 2>/dev/null
ip link delete flannel.1 2>/dev/null
systemctl start kubelet
```

## Related Documentation

- [Network Architecture](../architecture/network-architecture.md)
- [DNS Configuration](../components/infrastructure/dns.md)
- [Diagnostic Procedures](diagnostic-procedures.md)
