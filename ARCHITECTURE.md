# VMStation-Org Modular Architecture

**Date**: November 30, 2025  
**Version**: 1.0.0  
**Purpose**: Document the modular architecture, repository relationships, and deployment workflows

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Repository Structure](#repository-structure)
3. [Deployment Flow](#deployment-flow)
4. [Repository Dependencies](#repository-dependencies)
5. [Data Flow & Interactions](#data-flow--interactions)
6. [Network Architecture](#network-architecture)
7. [Technology Stack](#technology-stack)
8. [Operational Workflows](#operational-workflows)

---

## Architecture Overview

The vmstation-org architecture follows a **modular, repository-based design** with clear separation of concerns. Each repository has a specific purpose and well-defined boundaries, enabling parallel development, easier maintenance, and better GitOps integration.

### Design Principles

1. **Separation of Concerns**: Each repo handles one specific domain (infra, config, monitoring, apps)
2. **Clear Boundaries**: Explicit interfaces between repos (no circular dependencies)
3. **Centralized Documentation**: Single source of truth in `cluster-docs`
4. **Idempotent Operations**: All playbooks and scripts can be run repeatedly
5. **Progressive Enhancement**: Core platform first, then applications and monitoring
6. **Operations-Friendly**: Clear separation of setup vs operational tools

---

## Repository Structure

```
vmstation-org/
│
├── cluster-docs/                    # Centralized documentation hub
│   ├── architecture/               # Architecture diagrams and design
│   ├── components/                 # Per-component documentation
│   ├── deployment/                 # Deployment guides and procedures
│   ├── troubleshooting/            # Troubleshooting guides
│   ├── migration/                  # Migration documentation
│   └── CAPABILITY_PARITY_ANALYSIS.md
│
├── cluster-infra/                   # Infrastructure provisioning
│   ├── terraform/                  # IaC for infrastructure (future)
│   ├── ansible/                    # Cluster deployment playbooks
│   │   ├── playbooks/
│   │   │   ├── deploy-cluster.yaml
│   │   │   ├── reset-cluster.yaml
│   │   │   └── verify-cluster.yaml
│   │   └── roles/
│   │       └── preflight-rhel10/   # RHEL10 preparation
│   ├── scripts/
│   │   └── run-kubespray.sh        # Kubespray wrapper
│   └── inventory/                  # Cluster inventory
│
├── cluster-config/                  # System configuration management
│   ├── ansible/
│   │   ├── playbooks/
│   │   │   ├── site.yml            # Master playbook
│   │   │   ├── infrastructure-services.yml
│   │   │   ├── ntp-sync.yml
│   │   │   ├── syslog-server.yml
│   │   │   └── kerberos-setup.yml
│   │   └── roles/
│   │       ├── networking/         # Network configuration
│   │       ├── ssh/                # SSH hardening
│   │       ├── storage/            # Storage mounts
│   │       └── system/             # System settings
│   ├── hosts/                      # Per-host configurations
│   │   ├── masternode/
│   │   ├── storagenodet3500/
│   │   └── homelab/
│   ├── group/                      # Group variables
│   │   ├── common/
│   │   └── storage/
│   └── inventory/
│       └── hosts.ini
│
├── cluster-setup/                   # Initial setup and bootstrap
│   ├── bootstrap/                  # Initial node preparation
│   │   ├── install-dependencies.sh
│   │   ├── setup-ssh-keys.sh
│   │   ├── prepare-nodes.sh
│   │   └── verify-prerequisites.sh
│   ├── orchestration/              # Deployment orchestration
│   │   ├── deploy-wrapper.sh
│   │   └── quick-deploy.sh
│   ├── power-management/           # Power setup and config
│   │   ├── playbooks/
│   │   │   ├── setup-autosleep.yaml
│   │   │   ├── spin-down-cluster.yaml
│   │   │   └── deploy-event-wake.yaml
│   │   └── scripts/
│   │       └── vmstation-autosleep-monitor.sh
│   └── systemd/                    # Systemd unit files
│       └── vmstation-autosleep-monitor.service
│
├── cluster-monitor-stack/           # Monitoring and observability
│   ├── manifests/
│   │   ├── prometheus/             # Prometheus monitoring
│   │   ├── grafana/                # Grafana dashboards
│   │   ├── loki/                   # Loki log aggregation
│   │   ├── promtail/               # Promtail log shipping
│   │   └── exporters/              # Various exporters
│   │       ├── node-exporter/
│   │       ├── kube-state-metrics/
│   │       └── ipmi-exporter/
│   ├── ansible/playbooks/
│   │   ├── deploy-monitoring-stack.yaml
│   │   └── fix-loki-config.yaml
│   └── dashboards/                 # Grafana dashboards
│
├── cluster-application-stack/       # Application deployments
│   ├── manifests/
│   │   └── jellyfin/               # Jellyfin media server
│   ├── kustomize/                  # Kustomize overlays
│   │   └── overlays/
│   ├── helm-charts/                # Helm chart definitions
│   └── ansible/playbooks/
│       ├── jellyfin.yml
│       └── deploy-minecraft.yml
│
└── cluster-tools/                   # Operational utilities
    ├── validation/                 # Validation scripts
    │   ├── validate-cluster-health.sh
    │   ├── validate-monitoring-stack.sh
    │   ├── validate-network-connectivity.sh
    │   └── pre-deployment-checklist.sh
    ├── diagnostics/                # Diagnostic tools
    ├── remediation/                # Remediation scripts
    ├── power-management/           # Power operations
    │   ├── vmstation-event-wake.sh
    │   ├── vmstation-collect-wake-logs.sh
    │   └── check-power-state.sh
    ├── tests/
    │   ├── integration/            # Integration tests
    │   ├── drift-detection/        # Drift detection
    │   └── syntax/                 # Syntax validation
    └── lib/                        # Shared libraries
```

---

## Deployment Flow

### Phase 1: Infrastructure Provisioning

```
┌─────────────────────────────────────────────────────────────┐
│                     cluster-infra                            │
│                                                              │
│  1. Terraform (future)    →  Provision bare metal/VMs      │
│  2. Kubespray wrapper     →  Deploy Kubernetes cluster      │
│  3. preflight-rhel10      →  Prepare RHEL10 nodes          │
│                                                              │
│  Output: Running Kubernetes cluster with Calico CNI         │
└─────────────────────────────────────────────────────────────┘
                            ↓
```

### Phase 2: System Configuration

```
┌─────────────────────────────────────────────────────────────┐
│                     cluster-config                           │
│                                                              │
│  1. Apply baseline configs →  SSH hardening, sysctl, etc.   │
│  2. Configure networking   →  Network policies, DNS         │
│  3. Mount storage          →  NFS, local storage            │
│  4. Deploy infra services  →  NTP, Syslog, Kerberos        │
│                                                              │
│  Output: Configured cluster with baseline security          │
└─────────────────────────────────────────────────────────────┘
                            ↓
```

### Phase 3: Bootstrap & Setup

```
┌─────────────────────────────────────────────────────────────┐
│                     cluster-setup                            │
│                                                              │
│  1. Install dependencies   →  Ansible, Python packages      │
│  2. Setup SSH keys         →  Passwordless authentication   │
│  3. Prepare nodes          →  OS-level preparation          │
│  4. Configure power mgmt   →  Auto-sleep, WoL, systemd     │
│                                                              │
│  Output: Cluster ready for monitoring and applications      │
└─────────────────────────────────────────────────────────────┘
                            ↓
```

### Phase 4: Monitoring Stack

```
┌─────────────────────────────────────────────────────────────┐
│                  cluster-monitor-stack                       │
│                                                              │
│  1. Deploy Prometheus      →  Metrics collection            │
│  2. Deploy Grafana         →  Visualization dashboards      │
│  3. Deploy Loki/Promtail   →  Log aggregation              │
│  4. Deploy exporters       →  Node, IPMI, Kube-state       │
│                                                              │
│  Output: Full observability stack                           │
└─────────────────────────────────────────────────────────────┘
                            ↓
```

### Phase 5: Application Deployment

```
┌─────────────────────────────────────────────────────────────┐
│                cluster-application-stack                     │
│                                                              │
│  1. Deploy Jellyfin        →  Media server workload         │
│  2. Deploy other apps      →  Minecraft, etc.               │
│  3. Configure ingress      →  External access               │
│                                                              │
│  Output: Running application workloads                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
```

### Phase 6: Validation & Operations

```
┌─────────────────────────────────────────────────────────────┐
│                     cluster-tools                            │
│                                                              │
│  1. Validate deployment    →  Health checks, connectivity   │
│  2. Run integration tests  →  Sleep-wake cycle, etc.        │
│  3. Detect configuration drift                              │
│  4. Operational scripts    →  Power management, diagnostics │
│                                                              │
│  Output: Validated, operational cluster                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Repository Dependencies

### Dependency Graph

```
                   cluster-docs
                       │
                       │ (Documentation referenced by all)
                       │
                       ↓
              ┌────────────────────┐
              │   cluster-setup    │  ← Bootstrap first
              └────────────────────┘
                       │
                       ↓
              ┌────────────────────┐
              │   cluster-infra    │  ← Deploy cluster
              └────────────────────┘
                       │
                       ↓
              ┌────────────────────┐
              │   cluster-config   │  ← Configure nodes
              └────────────────────┘
                       │
              ┌────────┴────────┐
              ↓                 ↓
    ┌──────────────────┐  ┌─────────────────────────────┐
    │cluster-monitor-  │  │cluster-application-stack    │
    │stack             │  │                             │
    └──────────────────┘  └─────────────────────────────┘
              │                       │
              └───────────┬───────────┘
                          ↓
                  ┌───────────────┐
                  │cluster-tools  │  ← Validate & operate
                  └───────────────┘
```

### Dependency Matrix

| Repository | Depends On | Depended On By |
|------------|-----------|----------------|
| `cluster-docs` | None | All (documentation) |
| `cluster-setup` | None | `cluster-infra` |
| `cluster-infra` | `cluster-setup` | `cluster-config` |
| `cluster-config` | `cluster-infra` | `cluster-monitor-stack`, `cluster-application-stack` |
| `cluster-monitor-stack` | `cluster-config` | `cluster-tools` (validation) |
| `cluster-application-stack` | `cluster-config` | `cluster-tools` (validation) |
| `cluster-tools` | All | None (operational only) |

### Anti-Dependencies (What NOT to do)

❌ **Do NOT**:
- Have `cluster-infra` depend on `cluster-config` (infra first, config second)
- Have circular dependencies between any repos
- Have `cluster-tools` as a dependency (it's operational only)
- Deploy applications before monitoring stack (observability first)
- Skip `cluster-setup` bootstrap (required for all subsequent steps)

---

## Data Flow & Interactions

### Configuration Data Flow

```
┌──────────────────┐
│  cluster-config  │
│  hosts/          │  ← Per-host configs (fstab, sshd_config, etc.)
└────────┬─────────┘
         │
         ↓ Applied via Ansible
┌────────────────────────────────────────┐
│           Target Nodes                 │
│  • masternode (192.168.4.63)          │
│  • storagenodet3500 (192.168.4.61)    │
│  • homelab (192.168.4.62)             │
└────────────────────────────────────────┘
         │
         ↓ Metrics & Logs
┌──────────────────────┐
│  cluster-monitor-    │
│  stack               │  ← Collects metrics and logs
└──────────────────────┘
         │
         ↓ Visualization
┌─────────────────────────────────┐
│  Grafana Dashboards             │
│  http://192.168.4.63:30001      │
└─────────────────────────────────┘
```

### Power Management Data Flow

```
┌──────────────────┐
│  cluster-setup   │
│  power-mgmt/     │  ← Setup: systemd units, monitoring scripts
└────────┬─────────┘
         │
         ↓ Deployed to
┌────────────────────────────────────────┐
│           Target Nodes                 │
│  /etc/systemd/system/                  │
│  /usr/local/bin/vmstation-*.sh         │
└────────┬───────────────────────────────┘
         │
         ↓ Operational control
┌──────────────────┐
│  cluster-tools   │
│  power-mgmt/     │  ← Operations: wake, check status, logs
└──────────────────┘
```

### Application Deployment Flow

```
┌──────────────────────────┐
│  cluster-application-    │
│  stack                   │
│  manifests/jellyfin/     │
└────────┬─────────────────┘
         │
         ↓ kubectl apply / Ansible
┌────────────────────────────────────────┐
│     Kubernetes Cluster (masternode)    │
│     namespace: default                 │
└────────┬───────────────────────────────┘
         │
         ↓ Exposes service
┌─────────────────────────────────────┐
│  Jellyfin Application                │
│  http://192.168.4.63:30002          │
└─────────────────────────────────────┘
```

---

## Network Architecture

### Node Topology

```
                        Internet
                            │
                            │
                    ┌───────┴────────┐
                    │   Home Router  │
                    │  192.168.4.1   │
                    └───────┬────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
    ┌───────┴────────┐ ┌───┴──────────┐ ┌──┴───────────┐
    │  masternode    │ │storagenode   │ │  homelab     │
    │  192.168.4.63  │ │t3500         │ │192.168.4.62  │
    │  Debian 12     │ │192.168.4.61  │ │  RHEL 10     │
    │  Control Plane │ │  Debian 12   │ │  Compute     │
    │  Always-On     │ │  Worker      │ │  On-Demand   │
    └────────────────┘ │  Storage     │ └──────────────┘
                       │  Auto-Sleep  │
                       └──────────────┘
```

### Service Endpoints

| Service | URL | Port | Node |
|---------|-----|------|------|
| Kubernetes API | https://192.168.4.63:6443 | 6443 | masternode |
| Grafana | http://192.168.4.63:30001 | 30001 | masternode |
| Prometheus | http://192.168.4.63:30000 | 30000 | masternode |
| Jellyfin | http://192.168.4.63:30002 | 30002 | storagenodet3500 |
| Loki | http://192.168.4.63:3100 | 3100 | masternode |

### Network Policies

- **CNI**: Calico (via Kubespray)
- **Network Mode**: Overlay network with BGP routing
- **Pod CIDR**: 10.244.0.0/16 (default Calico)
- **Service CIDR**: 10.96.0.0/12 (Kubernetes default)
- **DNS**: CoreDNS (cluster-internal)

---

## Technology Stack

### Core Infrastructure

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Container Orchestration | Kubernetes | 1.29+ | Container orchestration |
| CNI | Calico | Latest | Network plugin |
| Deployment Tool | Kubespray | Latest | Kubernetes deployment |
| Configuration Management | Ansible | 2.15+ | System configuration |
| Scripting | Bash | 5.x | Automation scripts |
| Python | Python | 3.10+ | Ansible and tooling |

### Monitoring Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Metrics | Prometheus | 2.x | Metrics collection |
| Visualization | Grafana | 10.x | Dashboards |
| Logs | Loki | 2.x | Log aggregation |
| Log Shipping | Promtail | 2.x | Log forwarding |
| Node Metrics | Node Exporter | 1.x | System metrics |
| Kube Metrics | Kube-state-metrics | 2.x | K8s object metrics |
| IPMI Metrics | IPMI Exporter | Latest | Hardware metrics |

### Infrastructure Services

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Time Sync | Chrony/NTP | Latest | Time synchronization |
| Centralized Logging | Rsyslog | Latest | Syslog server |
| SSO | FreeIPA/Kerberos | Latest | Authentication |
| Power Management | systemd + custom | N/A | Auto-sleep, WoL |

### Application Layer

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Media Server | Jellyfin | 10.x | Media streaming |
| Package Manager | Helm | 3.x | K8s package manager |
| Template Engine | Kustomize | Latest | Manifest templates |

---

## Operational Workflows

### Daily Operations

```
┌─────────────────────────────────────────────────────────┐
│              Daily Operations Workflow                  │
└─────────────────────────────────────────────────────────┘

1. Check cluster health
   $ cd cluster-tools/validation
   $ ./validate-cluster-health.sh

2. Review Grafana dashboards
   → http://192.168.4.63:30001

3. Check for configuration drift
   $ cd cluster-tools/tests/drift-detection
   $ ./check-drift.sh

4. Review logs in Loki
   → http://192.168.4.63:3100
```

### Deployment Workflow

```
┌─────────────────────────────────────────────────────────┐
│           New Application Deployment                    │
└─────────────────────────────────────────────────────────┘

1. Add manifests to cluster-application-stack
   $ cd cluster-application-stack/manifests/
   $ mkdir my-app
   $ vim my-app/deployment.yaml

2. Create Ansible playbook (optional)
   $ cd ansible/playbooks/
   $ vim deploy-my-app.yml

3. Deploy application
   $ ansible-playbook deploy-my-app.yml
   # OR
   $ kubectl apply -f manifests/my-app/

4. Validate deployment
   $ cd cluster-tools/validation
   $ kubectl get pods -n my-app
   $ ./validate-network-connectivity.sh
```

### Configuration Update Workflow

```
┌─────────────────────────────────────────────────────────┐
│         Configuration Update Workflow                   │
└─────────────────────────────────────────────────────────┘

1. Update host config in cluster-config
   $ cd cluster-config/hosts/masternode/
   $ vim sshd_config

2. Apply configuration
   $ cd ansible/playbooks/
   $ ansible-playbook site.yml --tags ssh

3. Validate changes
   $ cd cluster-tools/validation
   $ ./validate-cluster-health.sh

4. Check for drift
   $ cd cluster-tools/tests/drift-detection
   $ ./check-drift.sh
```

### Power Management Workflow

```
┌─────────────────────────────────────────────────────────┐
│           Power Management Workflow                     │
└─────────────────────────────────────────────────────────┘

1. Setup auto-sleep (one-time)
   $ cd cluster-setup/power-management/playbooks
   $ ansible-playbook setup-autosleep.yaml

2. Wake node on-demand
   $ cd cluster-tools/power-management
   $ ./vmstation-event-wake.sh storagenodet3500

3. Check power state
   $ ./check-power-state.sh

4. Collect wake logs
   $ ./vmstation-collect-wake-logs.sh
```

### Troubleshooting Workflow

```
┌─────────────────────────────────────────────────────────┐
│            Troubleshooting Workflow                     │
└─────────────────────────────────────────────────────────┘

1. Run diagnostics
   $ cd cluster-tools/diagnostics
   $ ./diagnose-cluster.sh

2. Check component logs
   $ kubectl logs -n monitoring <pod-name>

3. Review Grafana dashboards
   → http://192.168.4.63:30001

4. Apply remediation if needed
   $ cd cluster-tools/remediation
   $ ./remediate-<issue>.sh

5. Validate fix
   $ cd cluster-tools/validation
   $ ./validate-cluster-health.sh
```

---

## Future Enhancements

### Planned CI/CD Integration

```
┌─────────────────────────────────────────────────────────┐
│              CI/CD Architecture (DRONE)                 │
└─────────────────────────────────────────────────────────┘

                    GitHub/GitLab
                         │
                         │ Webhook
                         ↓
                  ┌──────────────┐
                  │  DRONE CI    │
                  │  (masternode)│
                  └──────┬───────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ↓              ↓              ↓
    ┌─────────┐    ┌─────────┐   ┌─────────┐
    │  Lint   │    │  Test   │   │ Deploy  │
    │  Stage  │    │  Stage  │   │  Stage  │
    └─────────┘    └─────────┘   └─────────┘
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                 Kubernetes Cluster
```

### GitOps Workflow (Future)

```
┌─────────────────────────────────────────────────────────┐
│            GitOps Workflow (ArgoCD/Flux)                │
└─────────────────────────────────────────────────────────┘

    Git Repository (Source of Truth)
              │
              │ Sync
              ↓
        ┌──────────┐
        │ ArgoCD   │
        │ / Flux   │
        └────┬─────┘
             │
             ↓ Apply
      Kubernetes Cluster
```

---

## Maintenance & Support

### Version Control Strategy

- **Branching**: Feature branches → PR → Main
- **Tagging**: Semantic versioning (v1.0.0, v1.1.0, etc.)
- **Releases**: Tag releases after successful deployment
- **Rollback**: Use git tags to rollback to previous versions

### Backup Strategy

- **Cluster State**: etcd backups (automated)
- **Persistent Volumes**: Regular snapshots
- **Configuration**: Git repository (already versioned)
- **Documentation**: Git repository (already versioned)

### Disaster Recovery

1. Restore infrastructure (cluster-infra)
2. Restore configuration (cluster-config)
3. Restore monitoring (cluster-monitor-stack)
4. Restore applications (cluster-application-stack)
5. Validate (cluster-tools)

---

## Contact & Support

**Documentation Location**: `cluster-docs/`  
**Architecture Updates**: Submit PR to `cluster-docs/ARCHITECTURE.md`  
**Issue Tracking**: GitHub Issues in respective repositories

---

**Last Updated**: November 30, 2025  
**Version**: 1.0.0  
**Maintained By**: VMStation Infrastructure Team
