# VMStation Documentation

Comprehensive documentation for the VMStation homelab Kubernetes environment.

## Quick Links

- **[Getting Started](getting-started/README.md)** - Start here for new users
- **[Architecture](architecture/README.md)** - System design and components
- **[Deployment](deployment/README.md)** - Installation and deployment guides
- **[Operations](operations/README.md)** - Day-to-day operations
- **[Troubleshooting](troubleshooting/README.md)** - Problem solving guides

## Documentation Structure

### 📚 [Getting Started](getting-started/README.md)
- [Quick Start Guide](getting-started/quick-start.md)
- [Prerequisites](getting-started/prerequisites.md)
- [Installation](getting-started/installation.md)
- [First Deployment](getting-started/first-deployment.md)

### 🏗️ [Architecture](architecture/README.md)
- [Overview](architecture/overview.md)
- [Network Architecture](architecture/network-architecture.md)
- [Storage Architecture](architecture/storage-architecture.md)
- [Monitoring Architecture](architecture/monitoring-architecture.md)

### 🚀 [Deployment](deployment/README.md)
- [Cluster Deployment](deployment/cluster-deployment.md)
- [Kubespray Deployment](deployment/kubespray-deployment.md)
- [Monitoring Deployment](deployment/monitoring-deployment.md)
- [Application Deployment](deployment/application-deployment.md)
- [Infrastructure Services](deployment/infrastructure-services.md)
- [Deployment Reports](deployment/deployment-reports/)

### ⚙️ [Operations](operations/README.md)
- [Day-2 Operations](operations/day-2-operations.md)
- [Cluster Management](operations/cluster-management.md)
- [Backup & Restore](operations/backup-restore.md)
- [Scaling](operations/scaling.md)
- [Upgrades](operations/upgrades.md)
- [Power Management](operations/power-management.md)

### 🔧 [Troubleshooting](troubleshooting/README.md)
- [Common Issues](troubleshooting/common-issues.md)
- [Monitoring Issues](troubleshooting/monitoring-issues.md)
- [Grafana Fixes](troubleshooting/grafana-fixes.md)
- [Storage Issues](troubleshooting/storage-issues.md)
- [Network Issues](troubleshooting/network-issues.md)
- [Diagnostic Procedures](troubleshooting/diagnostic-procedures.md)

### 📦 [Components](components/README.md)
- **[Ansible](components/ansible/)** - Playbooks, roles, best practices
- **[Monitoring](components/monitoring/)** - Prometheus, Grafana, Loki
- **[Infrastructure](components/infrastructure/)** - NTP, Syslog, Kerberos, DNS
- **[Applications](components/applications/)** - Jellyfin
- **[Storage](components/storage/)** - Storage configuration

### 📖 [Reference](reference/README.md)
- [CLI Reference](reference/cli-reference.md)
- [API Reference](reference/api-reference.md)
- [Configuration Reference](reference/configuration-reference.md)
- [Glossary](reference/glossary.md)

### 💻 [Development](development/README.md)
- [Contributing](development/contributing.md)
- [Development Workflow](development/development-workflow.md)
- [Testing Guide](development/testing-guide.md)
- [AI Agent Implementation](development/ai-agent-implementation.md)
- [Coding Standards](development/coding-standards.md)

### 🗺️ [Roadmap](roadmap/README.md)
- [TODO](roadmap/todo.md)
- [Completed Features](roadmap/completed-features.md)
- [Future Plans](roadmap/future-plans.md)

### 🔄 [Migration](migration/README.md)
- [Migration Guide](migration/migration-guide.md)
- [Monorepo to Modular](migration/monorepo-to-modular.md)

## Cluster Overview

VMStation is a 3-node Kubernetes cluster:

| Node | IP | OS | Role | Status |
|------|----|----|------|--------|
| masternode | 192.168.4.63 | Debian 12 | Control Plane | Always-on |
| storagenodet3500 | 192.168.4.61 | Debian 12 | Worker, Storage | Auto-sleep |
| homelab | 192.168.4.62 | RHEL 10 | Compute | Auto-sleep |

### Service Access

| Service | URL |
|---------|-----|
| Grafana | http://192.168.4.63:30300 |
| Prometheus | http://192.168.4.63:30090 |
| Loki | http://192.168.4.63:31100 |

## Source Repository

Main code repository: [jjbly-vmstation/vmstation](https://github.com/jjbly-vmstation/vmstation)

## License

See [LICENSE](LICENSE) for details.
