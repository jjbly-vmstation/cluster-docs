# Network Architecture

Network design and configuration for VMStation.

## Network Overview

```
┌──────────────────────────────────────────────────────────┐
│                    Physical Network                       │
│                    192.168.4.0/24                         │
├──────────────────────────────────────────────────────────┤
│  masternode        storagenodet3500      homelab         │
│  192.168.4.63      192.168.4.61          192.168.4.62    │
│       │                  │                    │          │
│       └──────────────────┴────────────────────┘          │
│                         │                                │
│            ┌────────────┴────────────┐                   │
│            │                         │                   │
│     ┌──────┴──────┐           ┌──────┴──────┐           │
│     │ Pod Network │           │ Svc Network │           │
│     │ 10.244.0.0/16│          │10.96.0.0/12 │           │
│     └─────────────┘           └─────────────┘           │
└──────────────────────────────────────────────────────────┘
```

## Network Segments

| Network | CIDR | Purpose |
|---------|------|---------|
| Node Network | 192.168.4.0/24 | Physical host connectivity |
| Pod Network | 10.244.0.0/16 | Pod-to-pod communication |
| Service Network | 10.96.0.0/12 | ClusterIP services |

## Node IP Addresses

| Node | IP | MAC Address |
|------|----|-------------|
| masternode | 192.168.4.63 | - |
| storagenodet3500 | 192.168.4.61 | b8:ac:6f:7e:6c:9d |
| homelab | 192.168.4.62 | d0:94:66:30:d6:63 |

## CNI Configuration

### Flannel (Default)

Pod network overlay using VXLAN:

```yaml
net-conf.json: |
  {
    "Network": "10.244.0.0/16",
    "Backend": {
      "Type": "vxlan"
    }
  }
```

### Calico (Alternative)

Used with Kubespray deployment:
- Network policies support
- BGP peering capability
- Better performance

## Service Exposure

### NodePort Services

Exposed on ports 30000-32767 on all nodes:

| Service | NodePort | Internal Port |
|---------|----------|---------------|
| Grafana | 30300 | 3000 |
| Prometheus | 30090 | 9090 |
| Loki | 31100 | 3100 |
| Jellyfin | 30096 | 8096 |

### Access Pattern

```
External → Node:NodePort → Service → Pod
```

## DNS Configuration

### CoreDNS

Cluster DNS service:

| Property | Value |
|----------|-------|
| Service IP | 10.233.0.3 |
| Domain | cluster.local |
| Namespace | kube-system |

### Service Discovery

```
<service>.<namespace>.svc.cluster.local
```

Example: `prometheus.monitoring.svc.cluster.local`

### DNS Fix

If using nodelocaldns, update kubelet configuration:

```bash
# Change from nodelocaldns to CoreDNS
sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml
systemctl restart kubelet
```

## Required Ports

### Control Plane

| Port | Protocol | Component |
|------|----------|-----------|
| 6443 | TCP | kube-apiserver |
| 2379-2380 | TCP | etcd |
| 10250 | TCP | kubelet |
| 10251 | TCP | kube-scheduler |
| 10252 | TCP | kube-controller-manager |

### Worker Nodes

| Port | Protocol | Component |
|------|----------|-----------|
| 10250 | TCP | kubelet |
| 30000-32767 | TCP | NodePort services |

### Monitoring

| Port | Protocol | Component |
|------|----------|-----------|
| 9090 | TCP | Prometheus |
| 3000 | TCP | Grafana |
| 3100 | TCP | Loki |
| 9100 | TCP | Node Exporter |

## Network Policies

Currently no network policies enforced. For production:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

## Troubleshooting

### DNS Issues

```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Test DNS resolution
kubectl exec -it <pod> -- nslookup kubernetes.default
```

### Connectivity Issues

```bash
# Check pod networking
kubectl get pods -o wide

# Test pod connectivity
kubectl exec -it <pod> -- ping <other-pod-ip>
```

## Related Documentation

- [Architecture Overview](overview.md)
- [Network Issues](../troubleshooting/network-issues.md)
- [DNS Configuration](../components/infrastructure/dns.md)
