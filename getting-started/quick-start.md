# Quick Start Guide

Deploy the VMStation Kubernetes cluster using simplified modular deployment commands.

## Prerequisites

- Ansible installed on your control machine
- SSH access to all target nodes
- Inventory configured in /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml`

## Recommended Deployment (Kubespray)

```bash
# Clone repository
git clone https://github.com/JashandeepJustinBains/VMStation.git
cd VMStation

# Deploy full stack with Kubespray (RECOMMENDED)
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray  # Deploys cluster + monitoring + infrastructure

# Validate deployment
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Alternative: Legacy Debian-only Deployment

```bash
# Deploy to Debian nodes only (deprecated)
./deploy.sh reset
./deploy.sh setup
./deploy.sh debian
./deploy.sh monitoring
./deploy.sh infrastructure

# Validate deployment
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Step-by-Step Guide

### Step 1: Clean Slate (Optional)

```bash
./deploy.sh reset
```

Removes any previous Kubernetes installations and resets the cluster.

### Step 2: Deploy Kubernetes Cluster

```bash
./deploy.sh debian
```

Deploys the core Kubernetes cluster on Debian nodes:
- Control plane (masternode)
- Worker nodes (storagenodet3500)
- CNI networking (Flannel)

**Expected time:** 10-15 minutes

### Step 3: Deploy Monitoring Stack

```bash
./deploy.sh monitoring
```

Deploys the complete monitoring stack:
- **Prometheus** - http://192.168.4.63:30090
- **Grafana** - http://192.168.4.63:30300
- **Loki** - http://192.168.4.63:31100
- Node-exporter, Kube-state-metrics, Promtail

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

Configures automatic cluster sleep after 2 hours of inactivity.

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

# Run complete validation suite
./tests/test-complete-validation.sh
```

## Access URLs

| Service | URL | Description |
|---------|-----|-------------|
| Prometheus | http://192.168.4.63:30090 | Metrics collection and querying |
| Grafana | http://192.168.4.63:30300 | Dashboards and visualization |
| Loki | http://192.168.4.63:31100 | Log aggregation |

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
```

## Dry-Run Mode

Test what would happen without making changes:

```bash
./deploy.sh monitoring --check
./deploy.sh infrastructure --check
```

## Next Steps

- [Prerequisites](prerequisites.md) - Full requirements list
- [Installation Guide](installation.md) - Detailed installation
- [Troubleshooting](../troubleshooting/README.md) - Common issues
