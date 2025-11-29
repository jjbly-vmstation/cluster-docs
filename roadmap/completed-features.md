# Completed Features

Features and tasks completed in VMStation.

## Cluster Deployment

### ✅ Kubernetes Cluster (2024)

- 3-node cluster (masternode, storagenodet3500, homelab)
- kubeadm initialization
- Flannel/Calico CNI
- Worker node joining

### ✅ Kubespray Integration (January 2025)

- `scripts/run-kubespray.sh` script
- `ansible/roles/preflight-rhel10` role
- `ansible/playbooks/run-preflight-rhel10.yml` playbook
- Smoke tests

## Monitoring Stack

### ✅ Core Monitoring

- Prometheus deployment
- Grafana with pre-configured dashboards
- Loki log aggregation
- Promtail log shipping
- Node Exporter on all nodes
- Blackbox Exporter for probes
- kube-state-metrics

### ✅ Dashboard Improvements (October 2025)

- Enhanced Loki Logs & Aggregation dashboard
- Syslog Infrastructure Monitoring dashboard
- CoreDNS Performance & Health dashboard

### ✅ Monitoring Fixes (October 2025)

- Prometheus SecurityContext fix (runAsGroup)
- Loki frontend_worker disabled for single-instance
- Blackbox Exporter config fix
- Loki schema validation fix

## Infrastructure

### ✅ NTP/Chrony

- DaemonSet deployment
- Time sync across all nodes

### ✅ CoreDNS

- DNS resolution fix
- nodelocaldns removal

### ✅ Storage

- local-path-provisioner
- Persistent storage for monitoring

## Power Management

### ✅ Auto-Sleep

- Idle detection
- systemd timer
- Auto-suspend for worker nodes

### ✅ Wake-on-LAN

- WoL scripts
- Node wake automation
- MAC address configuration

## Automation

### ✅ deploy.sh

- Modular deployment commands
- `debian`, `rke2`, `monitoring`, `infrastructure`
- Reset and validation

### ✅ Diagnostic Scripts

- `diagnose-monitoring-stack.sh`
- `remediate-monitoring-stack.sh`
- `validate-monitoring-stack.sh`

### ✅ Test Suite

- Complete validation tests
- Individual component tests
- Idempotency testing

## Documentation

### ✅ Documentation Migration (2025)

- Migrated from vmstation monorepo
- Organized by topic
- Added navigation
- Created standards document

### ✅ Consolidated Guides

- Architecture documentation
- Troubleshooting guide
- Usage guide
- Rollback procedures

## Bug Fixes

### ✅ SSH Key Paths

- Changed to absolute paths
- Fixed inventory configuration

### ✅ homelab Kubelet

- Symlink creation for kubelet path
- Version compatibility

### ✅ Node Sleep Issues

- Stale counter handling
- Deployment coordination

## Related Documentation

- [TODO](todo.md)
- [Future Plans](future-plans.md)
