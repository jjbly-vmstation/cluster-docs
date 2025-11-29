# VMStation Documentation

A comprehensive documentation repository for the VMStation homelab Kubernetes environment with automated deployment, monitoring, and power management.

## Overview

VMStation provides a production-ready Kubernetes setup using **Kubespray** for deployment automation.

- **Primary Deployment**: Kubespray - Production-grade Kubernetes deployment
- **Legacy Option**: kubeadm (Debian nodes) - Deprecated but supported
- **Monitoring**: Prometheus, Grafana, Loki, exporters
- **Power Management**: Wake-on-LAN, auto-sleep
- **Infrastructure**: NTP/Chrony, Syslog, Kerberos (optional)

## Quick Start

```bash
# Clone the main repository
git clone https://github.com/jjbly-vmstation/vmstation.git
cd vmstation

# Deploy full stack with Kubespray (RECOMMENDED)
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray  # Deploys cluster + monitoring + infrastructure

# Validate deployment
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Documentation Structure

### 📚 Getting Started
- [Quick Start Guide](getting-started/quick-start.md) - Get up and running quickly
- [Prerequisites](getting-started/prerequisites.md) - Requirements and setup
- [Installation Guide](getting-started/installation.md) - Step-by-step installation
- [First Deployment](getting-started/first-deployment.md) - Your first cluster deployment

### 🏗️ Architecture
- [Overview](architecture/overview.md) - System architecture overview
- [Network Architecture](architecture/network-architecture.md) - Network topology and configuration
- [Storage Architecture](architecture/storage-architecture.md) - Storage setup and management
- [Monitoring Architecture](architecture/monitoring-architecture.md) - Monitoring stack design

### 🚀 Deployment
- [Cluster Deployment](deployment/cluster-deployment.md) - Deploying Kubernetes clusters
- [Kubespray Deployment](deployment/kubespray-deployment.md) - Kubespray-specific deployment
- [Monitoring Deployment](deployment/monitoring-deployment.md) - Deploying the monitoring stack
- [Application Deployment](deployment/application-deployment.md) - Deploying applications
- [Infrastructure Services](deployment/infrastructure-services.md) - Core infrastructure deployment
- **Deployment Reports**
  - [Final Status](deployment/deployment-reports/deployment-final-status.md)
  - [Report 2025-10-14](deployment/deployment-reports/deployment-report-20251014.md)

### 🔧 Operations
- [Day-2 Operations](operations/day-2-operations.md) - Ongoing operations guide
- [Cluster Management](operations/cluster-management.md) - Managing your cluster
- [Backup & Restore](operations/backup-restore.md) - Data backup and recovery
- [Scaling](operations/scaling.md) - Scaling your infrastructure
- [Upgrades](operations/upgrades.md) - Upgrading components
- [Power Management](operations/power-management.md) - Auto-sleep and WoL

### 🔍 Troubleshooting
- [Common Issues](troubleshooting/common-issues.md) - Frequently encountered problems
- [Monitoring Issues](troubleshooting/monitoring-issues.md) - Monitoring stack problems
- [Grafana Fixes](troubleshooting/grafana-fixes.md) - Grafana-specific solutions
- [Storage Issues](troubleshooting/storage-issues.md) - Storage troubleshooting
- [Network Issues](troubleshooting/network-issues.md) - Network diagnostics
- [Diagnostic Procedures](troubleshooting/diagnostic-procedures.md) - Diagnostic tools and procedures

### 🧩 Components
- **[Ansible](components/ansible/)**
  - [Playbooks](components/ansible/playbooks.md)
  - [Roles](components/ansible/roles.md)
  - [Best Practices](components/ansible/best-practices.md)
- **[Monitoring](components/monitoring/)**
  - [Prometheus](components/monitoring/prometheus.md)
  - [Grafana](components/monitoring/grafana.md)
  - [Loki](components/monitoring/loki.md)
  - [Dashboards](components/monitoring/dashboards.md)
  - [Exporters](components/monitoring/exporters.md)
- **[Infrastructure](components/infrastructure/)**
  - [NTP/Chrony](components/infrastructure/ntp-chrony.md)
  - [Syslog](components/infrastructure/syslog.md)
  - [Kerberos](components/infrastructure/kerberos.md)
  - [DNS](components/infrastructure/dns.md)
- **[Applications](components/applications/)**
  - [Jellyfin](components/applications/jellyfin.md)
- **[Storage](components/storage/)**
  - [Storage Setup](components/storage/storage-setup.md)

### 📖 Reference
- [CLI Reference](reference/cli-reference.md) - Command-line reference
- [API Reference](reference/api-reference.md) - API documentation
- [Configuration Reference](reference/configuration-reference.md) - Configuration options
- [Glossary](reference/glossary.md) - Terms and definitions

### 💻 Development
- [Contributing](development/contributing.md) - How to contribute
- [Development Workflow](development/development-workflow.md) - Development process
- [Testing Guide](development/testing-guide.md) - Testing procedures
- [AI Agent Implementation](development/ai-agent-implementation.md) - AI agent documentation
- [Coding Standards](development/coding-standards.md) - Code style guidelines

### 🗺️ Roadmap
- [TODO](roadmap/todo.md) - Current tasks and backlog
- [Completed Features](roadmap/completed-features.md) - What's done
- [Future Plans](roadmap/future-plans.md) - Upcoming features

### 🔄 Migration
- [Migration Guide](migration/migration-guide.md) - Migration procedures
- [Monorepo to Modular](migration/monorepo-to-modular.md) - Repository restructuring

## Cluster Architecture

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

## Node Specifications

| Node | IP | OS | Role | Auto-Sleep |
|------|----|----|------|------------|
| masternode | 192.168.4.63 | Debian 12 | Control Plane | ❌ Always-on |
| storagenodet3500 | 192.168.4.61 | Debian 12 | Worker, Storage | ✅ Yes |
| homelab | 192.168.4.62 | RHEL 10 | Compute | ✅ Yes |

## Monitoring Access

| Service | URL | Purpose |
|---------|-----|---------|
| Grafana | http://192.168.4.63:30300 | Dashboards |
| Prometheus | http://192.168.4.63:30090 | Metrics |
| Loki | http://192.168.4.63:31100 | Logs |

**Default Grafana credentials**: admin/admin (change on first login)

## Source Repository

Main code repository: [jjbly-vmstation/vmstation](https://github.com/jjbly-vmstation/vmstation)

## Documentation Standards

See [IMPROVEMENTS_AND_STANDARDS.md](IMPROVEMENTS_AND_STANDARDS.md) for documentation guidelines, best practices, and standards.

## Contributing

This is a personal homelab project, but suggestions and improvements are welcome!

1. Fork the documentation repository
2. Create a feature branch
3. Make your changes
4. Test documentation links
5. Submit a pull request

## License

See [LICENSE](LICENSE) file.
