# Installation Guide

This guide provides detailed installation instructions for VMStation.

## Overview

VMStation supports multiple deployment methods:

1. **Kubespray** (Recommended) - Production-grade Kubernetes deployment
2. **kubeadm** (Legacy) - Debian-only deployment
3. **RKE2** (Optional) - RHEL10 nodes

## Clone the Repository

```bash
git clone https://github.com/jjbly-vmstation/vmstation.git
cd vmstation
```

## Configure Inventory

Edit `ansible/inventory/hosts.yml` to match your environment:

```yaml
all:
  vars:
    kubernetes_version: "1.29"
    pod_network_cidr: "10.244.0.0/16"
    service_network_cidr: "10.96.0.0/12"
    control_plane_endpoint: "192.168.4.63:6443"
    cni_plugin: flannel

monitoring_nodes:
  hosts:
    masternode:
      ansible_host: 192.168.4.63
      ansible_connection: local

storage_nodes:
  hosts:
    storagenodet3500:
      ansible_host: 192.168.4.61
      ansible_user: root
      ansible_ssh_private_key_file: /root/.ssh/id_k3s
      mac_address: "b8:ac:6f:7e:6c:9d"

compute_nodes:
  hosts:
    homelab:
      ansible_host: 192.168.4.62
      ansible_user: jashandeepjustinbains
      ansible_become: true
      ansible_ssh_private_key_file: /root/.ssh/id_k3s
      mac_address: "d0:94:66:30:d6:63"
```

## Deployment Methods

### Method 1: Kubespray (Recommended)

Kubespray provides production-grade Kubernetes deployment.

#### Step 1: Run Preflight Checks

```bash
ansible-playbook -i ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml
```

This prepares RHEL10 nodes by:
- Installing required packages
- Configuring chrony
- Setting up firewall
- Loading kernel modules

#### Step 2: Stage Kubespray

```bash
./scripts/run-kubespray.sh
```

This clones Kubespray and creates a virtual environment.

#### Step 3: Deploy Cluster

```bash
./deploy.sh kubespray
```

Or manually:

```bash
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
```

### Method 2: Legacy kubeadm (Debian Only)

For Debian-only deployments:

```bash
# Reset any previous installation
./deploy.sh reset

# Deploy cluster
./deploy.sh debian
```

This runs:
- Phase 0: Install Kubernetes binaries
- Phase 1: System preparation
- Phase 2: CNI plugins installation
- Phase 3: Control plane initialization
- Phase 4: Worker node join
- Phase 5: Flannel CNI deployment
- Phase 6: Node validation
- Phase 7: Application deployment

### Method 3: RKE2 on RHEL10 (Optional)

For deploying RKE2 on the homelab node:

```bash
./deploy.sh rke2
```

This deploys:
- RKE2 server
- Canal CNI
- Node exporter for monitoring

## Deploy Monitoring Stack

```bash
./deploy.sh monitoring
```

Deploys:
- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **Loki** - Log aggregation
- **Promtail** - Log shipping
- **Node Exporter** - System metrics
- **Blackbox Exporter** - HTTP probes
- **Kube-state-metrics** - Kubernetes metrics

## Deploy Infrastructure Services

```bash
./deploy.sh infrastructure
```

Deploys:
- **Chrony** - Time synchronization
- **Syslog** - Centralized logging
- **FreeIPA** - Identity management (optional)

## Deploy Storage Class

For persistent volumes:

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p \
  '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Configure Auto-Sleep

```bash
./deploy.sh setup
```

This configures:
- Auto-sleep monitoring (2-hour idle threshold)
- Wake-on-LAN handlers
- Systemd timers

## Verification

### Check Cluster Status

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes
```

Expected output:
```
NAME               STATUS   ROLES           AGE   VERSION
masternode         Ready    control-plane   60m   v1.29.15
storagenodet3500   Ready    <none>          59m   v1.29.15
homelab            Ready    <none>          58m   v1.29.15
```

### Check Pods

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods -A
```

### Run Validation Suite

```bash
./tests/test-complete-validation.sh
```

## Post-Installation

### Access Monitoring

| Service | URL |
|---------|-----|
| Grafana | http://192.168.4.63:30300 |
| Prometheus | http://192.168.4.63:30090 |
| Loki | http://192.168.4.63:31100 |

### Configure Grafana

1. Open http://192.168.4.63:30300
2. Login with admin/admin
3. Change password when prompted
4. Datasources are pre-configured

## Troubleshooting Installation

### SSH Key Issues

Ensure absolute paths in inventory:

```yaml
ansible_ssh_private_key_file: /root/.ssh/id_k3s  # Correct
```

### Node Not Joining

Check kubelet logs:

```bash
ssh root@<node> "journalctl -u kubelet -n 50"
```

### Pods Stuck Pending

Deploy storage class if PVCs are pending:

```bash
kubectl get pvc -A
# If showing Pending, deploy local-path-provisioner
```

See [Troubleshooting](../troubleshooting/common-issues.md) for more.

## Next Steps

- [First Deployment](first-deployment.md) - Deploy your first app
- [Monitoring Setup](../components/monitoring/prometheus.md) - Configure monitoring
- [Operations Guide](../operations/cluster-management.md) - Day-2 operations
