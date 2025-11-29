# Architecture

This section covers the VMStation system architecture, including network topology, storage design, and monitoring infrastructure.

## Documents

- [Overview](overview.md) - High-level system architecture
- [Network Architecture](network-architecture.md) - Network topology and configuration
- [Storage Architecture](storage-architecture.md) - Storage setup and management
- [Monitoring Architecture](monitoring-architecture.md) - Observability stack design

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     VMStation Cluster                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Control Plane (masternode - 192.168.4.63)                 │   │
│  │                                                           │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │   │
│  │  │ API Server  │ │ Controller  │ │ Scheduler   │        │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘        │   │
│  │                                                           │   │
│  │  ┌─────────────────────────────────────────────────────┐  │   │
│  │  │ Monitoring Stack                                     │  │   │
│  │  │  ┌──────────┐ ┌─────────┐ ┌──────┐ ┌────────────┐  │  │   │
│  │  │  │Prometheus│ │ Grafana │ │ Loki │ │ Exporters  │  │  │   │
│  │  │  └──────────┘ └─────────┘ └──────┘ └────────────┘  │  │   │
│  │  └─────────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                     Kubernetes Network                           │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Worker Nodes                                                 ││
│  │                                                              ││
│  │  ┌────────────────────────┐  ┌────────────────────────┐    ││
│  │  │ storagenodet3500       │  │ homelab                │    ││
│  │  │ 192.168.4.61           │  │ 192.168.4.62           │    ││
│  │  │ ┌──────────────────┐   │  │ ┌──────────────────┐   │    ││
│  │  │ │ Jellyfin         │   │  │ │ Compute Workloads│   │    ││
│  │  │ │ Storage Services │   │  │ │ Node Exporter    │   │    ││
│  │  │ │ Node Exporter    │   │  │ └──────────────────┘   │    ││
│  │  │ │ Promtail         │   │  │ Auto-sleep: ✓          │    ││
│  │  │ └──────────────────┘   │  └────────────────────────┘    ││
│  │  │ Auto-sleep: ✓          │                                 ││
│  │  └────────────────────────┘                                 ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## Key Concepts

### Control Plane
The masternode hosts all control plane components and the monitoring stack. It is always-on to ensure cluster availability.

### Worker Nodes
Worker nodes run application workloads. They can be put to sleep automatically when idle to save power.

### Monitoring Stack
Centralized monitoring on the control plane collects metrics and logs from all nodes.

### Power Management
Auto-sleep with Wake-on-LAN enables energy-efficient operation while maintaining availability.
