# Architecture

System architecture documentation for VMStation.

## Overview

VMStation is a 3-node Kubernetes cluster designed for homelab use with:
- Production-grade deployment via Kubespray
- Complete monitoring stack
- Automated power management
- Infrastructure services

## Documents

- [Overview](overview.md) - High-level architecture
- [Network Architecture](network-architecture.md) - Networking design
- [Storage Architecture](storage-architecture.md) - Storage configuration
- [Monitoring Architecture](monitoring-architecture.md) - Observability stack

## Quick Reference

### Cluster Nodes

| Node | IP | Role | Auto-Sleep |
|------|----|------|------------|
| masternode | 192.168.4.63 | Control Plane | No |
| storagenodet3500 | 192.168.4.61 | Worker, Storage | Yes |
| homelab | 192.168.4.62 | Worker, Compute | Yes |

### Key Components

```
┌─────────────────────────────────────────────────────┐
│                    VMStation                        │
├─────────────────────────────────────────────────────┤
│  Monitoring        │  Infrastructure    │  Apps     │
│  - Prometheus      │  - NTP/Chrony      │  - Jellyfin
│  - Grafana         │  - Syslog          │           │
│  - Loki            │  - DNS (CoreDNS)   │           │
│  - Promtail        │  - Kerberos (opt)  │           │
├─────────────────────────────────────────────────────┤
│                  Kubernetes                         │
│  - Kubespray / kubeadm deployment                   │
│  - Calico / Flannel CNI                             │
│  - Local-path storage                               │
├─────────────────────────────────────────────────────┤
│                   Nodes                             │
│  - masternode (control plane, always-on)            │
│  - storagenodet3500 (storage, auto-sleep)           │
│  - homelab (compute, auto-sleep)                    │
└─────────────────────────────────────────────────────┘
```

## Related Documentation

- [Getting Started](../getting-started/README.md)
- [Deployment Guides](../deployment/README.md)
- [Components](../components/README.md)
