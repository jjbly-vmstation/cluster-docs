# Architecture Overview

High-level architecture of the VMStation Kubernetes cluster.

## Cluster Diagram

```
┌─────────────────────────────────────────────────────┐
│ Kubernetes Cluster (Kubespray)                      │
├─────────────────────────────────────────────────────┤
│ masternode (192.168.4.63) - Control Plane           │
│   - Kubernetes API Server                           │
│   - etcd                                            │
│   - Controller Manager                              │
│   - Scheduler                                       │
│   - Monitoring Stack (Prometheus, Grafana, Loki)    │
│   - Always-on                                       │
├─────────────────────────────────────────────────────┤
│ storagenodet3500 (192.168.4.61) - Worker            │
│   - Jellyfin (media streaming)                      │
│   - Storage workloads                               │
│   - Node Exporter, Promtail                         │
│   - Auto-sleep enabled                              │
├─────────────────────────────────────────────────────┤
│ homelab (192.168.4.62) - Worker (RHEL10)            │
│   - General compute workloads                       │
│   - Node Exporter, Promtail                         │
│   - Auto-sleep enabled                              │
└─────────────────────────────────────────────────────┘
```

## Node Specifications

| Node | IP | OS | Role | CPU | RAM | Disk | Auto-Sleep |
|------|----|----|------|-----|-----|------|------------|
| masternode | 192.168.4.63 | Debian 12 | Control Plane | 4 | 8GB | 100GB | No |
| storagenodet3500 | 192.168.4.61 | Debian 12 | Worker, Storage | 4 | 8GB | 500GB+ | Yes |
| homelab | 192.168.4.62 | RHEL 10 | Worker, Compute | 4 | 8GB | 100GB | Yes |

## Deployment Options

### Kubespray (Recommended)

Production-grade Kubernetes deployment using Ansible:
- Standard upstream Kubernetes
- Flexible CNI options (Calico, Flannel)
- Multi-node support
- Production-grade automation

### kubeadm (Legacy)

Debian-only deployment:
- Direct kubeadm initialization
- Flannel CNI
- Simpler but less flexible

### RKE2 (Alternative)

Rancher Kubernetes Engine for RHEL:
- Single binary deployment
- Built-in Canal CNI
- FIPS compliant option

## Component Stack

### Control Plane

| Component | Purpose |
|-----------|---------|
| kube-apiserver | API endpoint |
| etcd | State storage |
| kube-controller-manager | Controller loops |
| kube-scheduler | Pod scheduling |
| CoreDNS | Service discovery |

### Networking

| Component | Purpose |
|-----------|---------|
| Calico/Flannel | Pod networking |
| kube-proxy | Service networking |
| CoreDNS | DNS resolution |

### Monitoring

| Component | Purpose |
|-----------|---------|
| Prometheus | Metrics collection |
| Grafana | Visualization |
| Loki | Log aggregation |
| Promtail | Log shipping |
| Node Exporter | System metrics |

### Infrastructure

| Component | Purpose |
|-----------|---------|
| Chrony | Time synchronization |
| Syslog | Centralized logging |
| Kerberos | SSO (optional) |

## Data Flow

### Metrics Flow

```
Nodes (node-exporter) → Prometheus → Grafana
                              ↓
                        AlertManager
```

### Logs Flow

```
Nodes (promtail) → Loki → Grafana
```

### Service Discovery

```
Pods → CoreDNS → Services → Pods
```

## Power Management

### Auto-Sleep

Worker nodes automatically sleep after 2 hours of inactivity:

1. Monitor for idle (CPU, Jellyfin activity)
2. Cordon and drain pods
3. Suspend node
4. Wake via WoL when needed

### Wake-on-LAN

```bash
wakeonlan b8:ac:6f:7e:6c:9d  # storagenodet3500
wakeonlan d0:94:66:30:d6:63  # homelab
```

## Security Considerations

### Network Isolation

- Pod network: 10.244.0.0/16
- Service network: 10.96.0.0/12
- Node network: 192.168.4.0/24

### RBAC

- Default service accounts with minimal permissions
- Admin access via kubeconfig

### Secrets

- Kubernetes secrets for sensitive data
- Ansible vault for deployment credentials

## Related Documentation

- [Network Architecture](network-architecture.md)
- [Storage Architecture](storage-architecture.md)
- [Monitoring Architecture](monitoring-architecture.md)
