# DNS

CoreDNS configuration.

## Overview

CoreDNS provides cluster DNS resolution for service discovery.

## Deployment

Deployed by Kubernetes in kube-system namespace.

| Property | Value |
|----------|-------|
| Service IP | 10.233.0.3 |
| Port | 53 (UDP/TCP) |
| Domain | cluster.local |

## Service Discovery

### Pattern

```
<service>.<namespace>.svc.cluster.local
```

### Examples

```
prometheus.monitoring.svc.cluster.local
grafana.monitoring.svc.cluster.local
loki.monitoring.svc.cluster.local
```

## Check Status

### Pods

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

### Service

```bash
kubectl get svc -n kube-system kube-dns
```

### Test Resolution

```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
kubectl exec -it <pod> -- nslookup prometheus.monitoring.svc.cluster.local
```

## Troubleshooting

### DNS Not Resolving

1. **Check CoreDNS pods:**
   ```bash
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   kubectl logs -n kube-system -l k8s-app=kube-dns
   ```

2. **Check CoreDNS service:**
   ```bash
   kubectl get svc -n kube-system kube-dns
   ```

3. **Verify kubelet config:**
   ```bash
   cat /var/lib/kubelet/config.yaml | grep clusterDNS
   ```

### nodelocaldns Conflict

If nodelocaldns causes issues:

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

## Configuration

### CoreDNS ConfigMap

```bash
kubectl get configmap coredns -n kube-system -o yaml
```

### Corefile

```
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

### Custom DNS

Add custom entries:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  custom.server: |
    example.local:53 {
        hosts {
          192.168.4.100 myservice.example.local
        }
    }
```

## Metrics

CoreDNS exports Prometheus metrics:

```promql
coredns_dns_request_duration_seconds_count
coredns_dns_responses_total
coredns_cache_hits_total
```

## Dashboard

CoreDNS Performance dashboard available in Grafana.

## Related Documentation

- [Network Architecture](../../architecture/network-architecture.md)
- [Network Issues](../../troubleshooting/network-issues.md)
