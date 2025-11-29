# DNS

CoreDNS provides DNS resolution within the Kubernetes cluster.

## Overview

| Property | Value |
|----------|-------|
| Service | CoreDNS |
| IP | 10.233.0.3 |
| Namespace | kube-system |

## Purpose

CoreDNS enables:
- Service discovery (`svc.cluster.local`)
- Pod DNS resolution
- External DNS forwarding

## Configuration

### CoreDNS Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns
  namespace: kube-system
```

### Corefile

```
.:53 {
    errors
    health
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
    }
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

## Service Discovery

### Format

```
<service>.<namespace>.svc.cluster.local
```

### Examples

```bash
# Prometheus
prometheus.monitoring.svc.cluster.local

# Loki
loki.monitoring.svc.cluster.local

# Grafana
grafana.monitoring.svc.cluster.local
```

## Testing

### From Pod

```bash
kubectl run test --rm -it --image=busybox -- nslookup prometheus.monitoring.svc.cluster.local
```

### Check CoreDNS Pods

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

### Check DNS Service

```bash
kubectl get svc -n kube-system kube-dns
```

## Troubleshooting

### DNS Resolution Failed

```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check logs
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### Restart CoreDNS

```bash
kubectl rollout restart deployment/coredns -n kube-system
```

### Check Kubelet DNS Config

```bash
ssh <node> "cat /var/lib/kubelet/config.yaml | grep clusterDNS"
```

Should show: `10.233.0.3`

### Fix Wrong DNS

```bash
ssh <node> "sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml"
ssh <node> "systemctl restart kubelet"
```

## CoreDNS Metrics

Available via Prometheus:
- coredns_dns_requests_total
- coredns_dns_responses_total
- coredns_forward_requests_total
- coredns_cache_hits_total

## Related Documentation

- [Network Architecture](../../architecture/network-architecture.md)
- [Network Issues](../../troubleshooting/network-issues.md)
