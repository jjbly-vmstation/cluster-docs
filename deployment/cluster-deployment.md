# Cluster Deployment Guide

This guide covers deploying Kubernetes clusters using VMStation.

## Deployment Methods

VMStation supports three deployment methods:

1. **Kubespray** (Recommended) - Production-grade deployment
2. **kubeadm** (Legacy) - Debian-only deployment
3. **RKE2** (Optional) - RHEL10 nodes

## Prerequisites

### Pre-Deployment Checklist

```bash
# Run automated checklist
./tests/pre-deployment-checklist.sh
```

Manual checks:
- [ ] All nodes accessible via SSH
- [ ] Ansible installed on control node
- [ ] Inventory configured
- [ ] Storage directories created
- [ ] Time synchronized

## Method 1: Kubespray (Recommended)

### Step 1: Reset Previous Installation

```bash
./deploy.sh reset
```

### Step 2: Setup Auto-Sleep

```bash
./deploy.sh setup
```

### Step 3: Deploy with Kubespray

```bash
./deploy.sh kubespray
```

This single command:
- Prepares RHEL10 nodes
- Runs Kubespray cluster.yml
- Deploys monitoring stack
- Deploys infrastructure services

### Manual Kubespray Deployment

```bash
# Preflight checks
ansible-playbook -i ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml

# Stage Kubespray
./scripts/run-kubespray.sh

# Deploy cluster
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
```

## Method 2: Legacy kubeadm

### Deploy Debian Cluster

```bash
./deploy.sh debian
```

**Phases executed:**
1. Phase 0: Install Kubernetes binaries
2. Phase 1: System preparation
3. Phase 2: CNI plugins installation
4. Phase 3: Control plane initialization
5. Phase 4: Worker node join
6. Phase 5: Flannel CNI deployment
7. Phase 6: Cluster validation
8. Phase 7: Application deployment

### Deploy Monitoring

```bash
./deploy.sh monitoring
```

### Deploy Infrastructure

```bash
./deploy.sh infrastructure
```

## Method 3: RKE2 on RHEL10

### Deploy RKE2

```bash
./deploy.sh rke2
```

**What happens:**
- RKE2 server installation
- Canal CNI configuration
- Service enablement
- Kubeconfig retrieval

### Access RKE2 Cluster

```bash
export KUBECONFIG=ansible/artifacts/homelab-rke2-kubeconfig.yaml
kubectl get nodes
```

## Deployment Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--yes` | Skip confirmations | `./deploy.sh debian --yes` |
| `--check` | Dry-run mode | `./deploy.sh monitoring --check` |
| `--with-rke2` | Include RKE2 | `./deploy.sh all --with-rke2` |
| `--log-dir` | Custom log directory | `./deploy.sh debian --log-dir=/tmp/logs` |

## All-in-One Deployment

```bash
./deploy.sh all --with-rke2 --yes
```

**Note:** Still need to run separately:
```bash
./deploy.sh monitoring
./deploy.sh infrastructure
./deploy.sh setup
```

## Verification

### Check Nodes

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes
```

Expected:
```
NAME               STATUS   ROLES           AGE   VERSION
masternode         Ready    control-plane   10m   v1.29.15
storagenodet3500   Ready    <none>          9m    v1.29.15
homelab            Ready    <none>          8m    v1.29.15
```

### Check Pods

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf get pods -A
```

### Run Validation Suite

```bash
./tests/test-complete-validation.sh
```

## Idempotency

Deployments are idempotent - safe to run multiple times:

```bash
# Test idempotency
./tests/test-idempotence.sh 3
```

## Reset Cluster

```bash
./deploy.sh reset
```

**Actions:**
- kubeadm reset on all nodes
- RKE2 uninstall on homelab
- CNI cleanup
- iptables flush
- Directory cleanup

## Troubleshooting

### SSH Key Issues

Ensure absolute paths in inventory:
```yaml
ansible_ssh_private_key_file: /root/.ssh/id_k3s  # Correct
```

### Node Not Joining

```bash
# Check kubelet logs
ssh root@<node> "journalctl -u kubelet -n 50"

# Check join token
kubectl --kubeconfig=/etc/kubernetes/admin.conf -n kube-system get secrets | grep bootstrap
```

### Pods Stuck Pending

```bash
# Check if storage class exists
kubectl get sc

# Deploy local-path-provisioner if needed
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml
```

### View Deployment Logs

```bash
ls -lh ansible/artifacts/
less ansible/artifacts/deploy-debian.log
```

## Related Documentation

- [Kubespray Deployment](kubespray-deployment.md)
- [Monitoring Deployment](monitoring-deployment.md)
- [Prerequisites](../getting-started/prerequisites.md)
- [Troubleshooting](../troubleshooting/common-issues.md)
