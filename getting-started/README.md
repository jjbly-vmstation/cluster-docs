# Getting Started

Welcome to VMStation! This section will help you get up and running with your homelab Kubernetes environment.

## Quick Start

For experienced users, jump straight to the [Quick Start Guide](quick-start.md).

## New Users

If you're new to VMStation, follow these guides in order:

1. **[Prerequisites](prerequisites.md)** - What you need before starting
2. **[Installation](installation.md)** - How to install and configure
3. **[First Deployment](first-deployment.md)** - Deploy your first workload

## What is VMStation?

VMStation is a production-ready homelab Kubernetes environment featuring:

- **Automated Deployment** - Kubespray and kubeadm deployment options
- **Complete Monitoring** - Prometheus, Grafana, Loki stack
- **Power Management** - Wake-on-LAN and auto-sleep
- **Infrastructure Services** - NTP, Syslog, optional Kerberos

## Cluster Architecture

```
┌─────────────────────────────────────────────────────┐
│ Kubernetes Cluster (Kubespray)                      │
├─────────────────────────────────────────────────────┤
│ masternode (192.168.4.63) - Control Plane           │
│   - Kubernetes API Server                           │
│   - Monitoring Stack (Prometheus, Grafana, Loki)    │
│   - Always-on                                       │
├─────────────────────────────────────────────────────┤
│ storagenodet3500 (192.168.4.61) - Worker            │
│   - Jellyfin (media streaming)                      │
│   - Storage workloads                               │
│   - Auto-sleep enabled                              │
├─────────────────────────────────────────────────────┤
│ homelab (192.168.4.62) - Worker (RHEL10)            │
│   - General compute workloads                       │
│   - Node Exporter                                   │
│   - Auto-sleep enabled                              │
└─────────────────────────────────────────────────────┘
```

## Service URLs

After deployment, access these services:

| Service | URL | Description |
|---------|-----|-------------|
| Grafana | http://192.168.4.63:30300 | Dashboards and visualization |
| Prometheus | http://192.168.4.63:30090 | Metrics collection |
| Loki | http://192.168.4.63:31100 | Log aggregation |

## Next Steps

- [Quick Start Guide](quick-start.md) - Deploy in minutes
- [Architecture Overview](../architecture/overview.md) - Understand the system
- [Troubleshooting](../troubleshooting/README.md) - Common issues

## Getting Help

1. Check the [Troubleshooting Guide](../troubleshooting/README.md)
2. Run diagnostic scripts: `./scripts/diagnose-monitoring-stack.sh`
3. Review deployment logs in `ansible/artifacts/`
