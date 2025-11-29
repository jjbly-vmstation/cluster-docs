# Network Architecture

VMStation uses a flat network topology with Kubernetes overlay networking.

## Physical Network

### Topology

```
┌─────────────────────────────────────────────────────────────────┐
│                    Home Network (192.168.4.0/24)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│     ┌─────────────┐                                             │
│     │   Router    │ (Gateway: 192.168.4.1)                      │
│     └──────┬──────┘                                             │
│            │                                                     │
│     ┌──────┴──────┐                                             │
│     │   Switch    │                                             │
│     └──────┬──────┘                                             │
│            │                                                     │
│     ┌──────┼──────────────────────────┐                         │
│     │      │                          │                         │
│     ▼      ▼                          ▼                         │
│  ┌──────┐ ┌──────────────┐  ┌────────────────┐                 │
│  │master│ │storagenodet3500│  │    homelab     │                 │
│  │ node │ │              │  │                │                 │
│  │.4.63 │ │    .4.61     │  │     .4.62      │                 │
│  └──────┘ └──────────────┘  └────────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### IP Addressing

| Node | IP Address | MAC Address | Gateway |
|------|------------|-------------|---------|
| masternode | 192.168.4.63 | - | 192.168.4.1 |
| storagenodet3500 | 192.168.4.61 | b8:ac:6f:7e:6c:9d | 192.168.4.1 |
| homelab | 192.168.4.62 | d0:94:66:30:d6:63 | 192.168.4.1 |

### DNS

- Internal DNS: CoreDNS (10.233.0.3)
- External DNS: Via router
- Service discovery: `.svc.cluster.local`

## Kubernetes Networking

### Network Configuration

```yaml
Pod Network CIDR: 10.244.0.0/16
Service CIDR: 10.96.0.0/12
DNS Service IP: 10.233.0.3
```

### CNI: Calico

Calico provides:
- Pod networking
- Network policies
- IP-in-IP encapsulation

```
┌────────────────────────────────────────────────────────────────┐
│                    Kubernetes Network                           │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Calico CNI                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   │
│  │  │ Pod Network │  │  Policies   │  │   Routing   │     │   │
│  │  │ 10.244.0.0  │  │             │  │             │     │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Service Network                        │   │
│  │  ClusterIP: 10.96.0.0/12                                │   │
│  │  NodePort: 30000-32767                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### Service Types

| Type | Port Range | Use Case |
|------|------------|----------|
| ClusterIP | 10.96.x.x | Internal services |
| NodePort | 30000-32767 | External access |
| LoadBalancer | N/A | Not configured |

## Exposed Services

### Monitoring Stack (NodePort)

| Service | Port | Internal Endpoint |
|---------|------|-------------------|
| Grafana | 30300 | grafana.monitoring:3000 |
| Prometheus | 30090 | prometheus.monitoring:9090 |
| Loki | 31100 | loki.monitoring:3100 |

### Applications

| Service | Port | Description |
|---------|------|-------------|
| Jellyfin | 30096 | Media streaming |

### Access Examples

```bash
# Grafana
curl http://192.168.4.63:30300/api/health

# Prometheus
curl http://192.168.4.63:30090/-/healthy

# Loki
curl http://192.168.4.63:31100/ready
```

## Firewall Configuration

### Control Plane (masternode)

```bash
# Kubernetes API
6443/tcp

# etcd
2379-2380/tcp

# Kubelet
10250/tcp

# kube-scheduler
10251/tcp

# kube-controller-manager
10252/tcp

# NodePort services
30000-32767/tcp
```

### Worker Nodes

```bash
# Kubelet
10250/tcp

# NodePort services
30000-32767/tcp

# Flannel VXLAN
8472/udp

# Node Exporter (optional)
9100/tcp
```

## Wake-on-LAN

### Configuration

Wake-on-LAN is enabled at BIOS level for worker nodes.

```bash
# Wake storagenodet3500
wakeonlan -i 192.168.4.255 b8:ac:6f:7e:6c:9d

# Wake homelab
wakeonlan -i 192.168.4.255 d0:94:66:30:d6:63
```

### Network Requirements

- Nodes must be on same broadcast domain
- Magic packets sent to broadcast address
- Switch must forward broadcast traffic

## DNS Resolution

### Internal (Kubernetes)

```
Service format: <service>.<namespace>.svc.cluster.local

Examples:
- prometheus.monitoring.svc.cluster.local
- grafana.monitoring.svc.cluster.local
- loki.monitoring.svc.cluster.local
```

### CoreDNS Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
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

## Network Troubleshooting

### Check Pod Networking

```bash
# Get pod IPs
kubectl get pods -o wide -A

# Test pod connectivity
kubectl exec -it <pod> -- ping <other-pod-ip>
```

### Check Service Endpoints

```bash
# List endpoints
kubectl get endpoints -n monitoring

# Test service DNS
kubectl run test --rm -it --image=busybox -- nslookup prometheus.monitoring.svc.cluster.local
```

### Check CoreDNS

```bash
# CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# DNS service
kubectl get svc -n kube-system kube-dns
```

### Check Calico

```bash
# Calico pods
kubectl get pods -n kube-system -l k8s-app=calico-node

# Calico node status
kubectl exec -n kube-system calico-node-xxxxx -- calico-node -bird-ready
```

## Related Documentation

- [Architecture Overview](overview.md)
- [Storage Architecture](storage-architecture.md)
- [DNS Infrastructure](../components/infrastructure/dns.md)
