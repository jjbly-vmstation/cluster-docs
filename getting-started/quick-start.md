# VMStation Quick Start Guide

This guide shows you how to quickly deploy the VMStation Kubernetes cluster using the simplified modular deployment commands.

## Prerequisites

- Ansible installed on your control machine
- SSH access to all target nodes
- Inventory configured in `ansible/inventory/hosts.yml`

See [Prerequisites](prerequisites.md) for detailed requirements.

## Simplified Deployment Workflow

### Step 1: Clean Slate (Optional)

```bash
./deploy.sh reset
```

Removes any previous Kubernetes installations and resets the cluster to a clean state.

### Step 2: Deploy Kubernetes Cluster

**Option A: Kubespray (RECOMMENDED)**

```bash
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray  # Deploys cluster + monitoring + infrastructure
```

**Option B: Legacy Debian-only**

```bash
./deploy.sh debian
```

Deploys the core Kubernetes cluster (kubeadm) on Debian nodes:
- Control plane (masternode)
- Worker nodes (storagenodet3500)
- CNI networking (Flannel)

**Expected time:** 10-15 minutes

### Step 3: Deploy Monitoring Stack

```bash
./deploy.sh monitoring
```

Deploys the complete monitoring and observability stack:
- **Prometheus** (metrics) - http://192.168.4.63:30090
- **Grafana** (dashboards) - http://192.168.4.63:30300
- **Loki** (logs) - http://192.168.4.63:31100
- Node-exporter, Kube-state-metrics, IPMI-exporter, Promtail

**Expected time:** 5-10 minutes

### Step 4: Deploy Infrastructure Services

```bash
./deploy.sh infrastructure
```

Deploys core infrastructure services:
- **NTP/Chrony** - Cluster-wide time synchronization
- **Syslog Server** - Centralized log aggregation
- **FreeIPA/Kerberos** - Identity management (optional)

**Expected time:** 3-5 minutes

### Step 5: Setup Auto-Sleep (Optional)

```bash
./deploy.sh setup
```

Configures automatic cluster sleep after 2 hours of inactivity to save power.

**Expected time:** 1-2 minutes

## Validation

After deployment, verify everything is working:

```bash
# Check cluster nodes
kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes

# Check monitoring pods
kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods -n monitoring

# Check infrastructure pods
kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods -n infrastructure

# Validate time synchronization
./tests/validate-time-sync.sh

# Run complete validation suite
./tests/test-complete-validation.sh
```

## Dry-Run Mode

Test what would happen without making changes:

```bash
./deploy.sh monitoring --check
./deploy.sh infrastructure --check
```

## Common Commands

```bash
# Get help
./deploy.sh help

# Deploy just monitoring
./deploy.sh monitoring

# Deploy just infrastructure
./deploy.sh infrastructure

# Reset everything
./deploy.sh reset

# Full deployment (non-interactive)
./deploy.sh all --with-rke2 --yes
./deploy.sh monitoring --yes
./deploy.sh infrastructure --yes
./deploy.sh setup --yes
```

## Access URLs

After deployment, access the services at:

| Service | URL | Description |
|---------|-----|-------------|
| Prometheus | http://192.168.4.63:30090 | Metrics collection and querying |
| Grafana | http://192.168.4.63:30300 | Dashboards and visualization |
| Loki | http://192.168.4.63:31100 | Log aggregation |

## Troubleshooting

If you encounter issues:

1. Check logs in `ansible/artifacts/`
2. Review the detailed runbook: [Deployment Guide](../deployment/cluster-deployment.md)
3. Run validation tests: `./tests/test-modular-deployment.sh`
4. Check pod status: `kubectl get pods -A`

## Next Steps

- [Installation Guide](installation.md) - Complete installation details
- [First Deployment](first-deployment.md) - Deploy your first application
- [Monitoring Setup](../components/monitoring/prometheus.md) - Configure monitoring
