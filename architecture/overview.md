# Architecture Overview

VMStation is a homelab Kubernetes environment designed for reliability, observability, and energy efficiency.

## System Design Goals

1. **Production-Ready** - Enterprise-grade Kubernetes with Kubespray
2. **Observable** - Complete monitoring with Prometheus, Grafana, Loki
3. **Energy Efficient** - Auto-sleep with Wake-on-LAN
4. **Maintainable** - Infrastructure as Code with Ansible
5. **Flexible** - Support for multiple deployment methods

## Cluster Topology

### Three-Node Architecture

```
┌─────────────────────────────────────────────────────┐
│ Kubernetes Cluster (Kubespray)                      │
├─────────────────────────────────────────────────────┤
│ masternode (192.168.4.63) - Control Plane           │
│   - Kubernetes API Server                            │
│   - Monitoring Stack (Prometheus, Grafana, Loki)    │
│   - Always-on                                        │
├─────────────────────────────────────────────────────┤
│ storagenodet3500 (192.168.4.61) - Worker            │
│   - Jellyfin (media streaming)                       │
│   - Storage workloads                                │
│   - Auto-sleep enabled                               │
├─────────────────────────────────────────────────────┤
│ homelab (192.168.4.62) - Worker (RHEL10)            │
│   - General compute workloads                        │
│   - Node Exporter                                    │
│   - Auto-sleep enabled                               │
└─────────────────────────────────────────────────────┘
```

### Node Roles

| Node | IP | OS | Role | Special Features |
|------|----|----|------|-----------------|
| masternode | 192.168.4.63 | Debian 12 | Control Plane | Always-on, hosts monitoring |
| storagenodet3500 | 192.168.4.61 | Debian 12 | Worker | Storage, media serving |
| homelab | 192.168.4.62 | RHEL 10 | Worker | General compute |

## Component Architecture

### Kubernetes Layer

```
┌──────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                         │
├──────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐ │
│  │  kube-apiserver │  │ kube-scheduler │  │ kube-controller│ │
│  └────────────────┘  └────────────────┘  └────────────────┘ │
├──────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐                      │
│  │     etcd       │  │   CoreDNS      │                      │
│  └────────────────┘  └────────────────┘                      │
├──────────────────────────────────────────────────────────────┤
│  ┌────────────────────────────────────────────────────────┐  │
│  │                   Calico CNI                            │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### Monitoring Stack

```
┌──────────────────────────────────────────────────────────────┐
│                    Monitoring Stack                           │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────┐     ┌──────────────────────────────┐   │
│  │   Prometheus    │────▶│         Grafana              │   │
│  │  (metrics)      │     │     (visualization)          │   │
│  └─────────────────┘     └──────────────────────────────┘   │
│          ▲                            │                      │
│          │                            │                      │
│  ┌───────┴───────┐          ┌────────▼────────┐            │
│  │   Exporters   │          │      Loki       │            │
│  │ • Node        │          │    (logs)       │            │
│  │ • Blackbox    │          └────────▲────────┘            │
│  │ • IPMI        │                   │                      │
│  │ • kube-state  │          ┌────────┴────────┐            │
│  └───────────────┘          │    Promtail     │            │
│                             │  (log shipper)  │            │
│                             └─────────────────┘            │
└──────────────────────────────────────────────────────────────┘
```

### Infrastructure Services

```
┌──────────────────────────────────────────────────────────────┐
│                  Infrastructure Services                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │  NTP/Chrony     │  │   Syslog        │  │  Kerberos    │ │
│  │  (time sync)    │  │  (central logs) │  │  (optional)  │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

## Data Flow

### Metrics Flow

```
Nodes (node-exporter) ──▶ Prometheus ──▶ Grafana Dashboards
Kubernetes (kube-state) ─┘
Applications ────────────┘
```

### Logs Flow

```
Containers ──▶ Promtail ──▶ Loki ──▶ Grafana Explore
System logs ─┘
```

### Wake-on-LAN Flow

```
Timer/Manual ──▶ WoL Script ──▶ Magic Packet ──▶ Node BIOS ──▶ Boot
                      │
                      ▼
              SSH Health Check ──▶ Uncordon Node ──▶ Pod Scheduling
```

## Deployment Methods

### Kubespray (Recommended)

```
ansible ──▶ preflight ──▶ kubespray ──▶ monitoring ──▶ infrastructure
```

### Legacy kubeadm

```
deploy.sh debian ──▶ kubeadm init ──▶ join workers ──▶ CNI ──▶ apps
```

### RKE2 (RHEL10)

```
deploy.sh rke2 ──▶ RKE2 install ──▶ start service ──▶ configure
```

## High Availability Considerations

### Current State (Single Control Plane)

- Single control plane node (masternode)
- Worker nodes can fail without cluster impact
- Monitoring data on control plane

### Future HA Options

- Add additional control plane nodes
- External etcd cluster
- Distributed monitoring with Thanos

## Security Architecture

### Network Security

- Calico CNI with network policies
- NodePort services for external access
- Firewall rules on each node

### Authentication

- Kubernetes RBAC
- Optional Kerberos/FreeIPA integration
- Grafana anonymous access (Viewer role)

### Secrets Management

- Ansible Vault for deployment secrets
- Kubernetes Secrets for runtime
- No secrets in Git

## Related Documentation

- [Network Architecture](network-architecture.md)
- [Storage Architecture](storage-architecture.md)
- [Monitoring Architecture](monitoring-architecture.md)
