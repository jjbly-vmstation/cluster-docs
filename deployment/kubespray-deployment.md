# Kubespray Deployment

Deploy Kubernetes using Kubespray for production-grade clusters.

## Overview

Kubespray is an Ansible-based Kubernetes deployment tool offering:
- Standard upstream Kubernetes
- Multiple CNI options (Calico, Flannel, Cilium)
- Multi-node cluster support
- Production-grade automation

## Quick Start

```bash
# Prepare RHEL10 node
ansible-playbook -i /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml

# Stage Kubespray
./scripts/run-kubespray.sh

# Deploy cluster
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml
```

## Step 1: Preflight Checks

Run preflight on RHEL10 nodes:

```bash
ansible-playbook -i /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml
```

This will:
- Install Python3 and required packages
- Configure chrony (time sync)
- Setup sudoers for ansible user
- Open firewall ports
- Configure SELinux (permissive)
- Load kernel modules
- Apply sysctl settings
- Disable swap

## Step 2: Stage Kubespray

```bash
./scripts/run-kubespray.sh
```

This will:
- Clone Kubespray into `.cache/kubespray`
- Create Python virtual environment
- Install Kubespray requirements
- Create inventory template

## Step 3: Configure Inventory

Edit `.cache/kubespray/inventory/mycluster/inventory.ini`:

```ini
[all]
masternode ansible_host=192.168.4.63 ansible_user=root
storagenodet3500 ansible_host=192.168.4.61 ansible_user=root
homelab ansible_host=192.168.4.62 ansible_user=jashandeepjustinbains

[kube_control_plane]
masternode

[kube_node]
storagenodet3500
homelab

[etcd]
masternode

[k8s_cluster:children]
kube_control_plane
kube_node
```

## Step 4: Customize Cluster Variables

Edit `.cache/kubespray/inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml`:

```yaml
kube_version: v1.29.6
kube_network_plugin: calico  # or flannel, weave, cilium
container_manager: containerd
```

## Step 5: Deploy Cluster

```bash
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
```

Expected time: 30-60 minutes

## Step 6: Access Cluster

```bash
# Copy kubeconfig from control plane
scp root@192.168.4.63:/etc/kubernetes/admin.conf ~/.kube/config

# Verify
kubectl get nodes
```

## Kubespray vs Other Options

| Feature | Kubespray | kubeadm | RKE2 |
|---------|-----------|---------|------|
| Deployment | Ansible | Manual | Binary |
| CNI Options | Multiple | Manual | Canal |
| Multi-node | Yes | Yes | Yes |
| RHEL Support | Native | Manual | Native |
| Complexity | Medium | Low | Low |
| Production Ready | Yes | Depends | Yes |

## Upgrade Cluster

```bash
cd .cache/kubespray
source .venv/bin/activate

# Update version in group_vars
# Edit k8s-cluster.yml: kube_version: v1.30.0

ansible-playbook -i inventory/mycluster/inventory.ini upgrade-cluster.yml -b
```

## Add Node

1. Add to inventory
2. Run cluster.yml with `--limit <new-node>`

```bash
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --limit newnode
```

## Remove Node

```bash
ansible-playbook -i inventory/mycluster/inventory.ini remove-node.yml -b -e node=<nodename>
```

## Reset Cluster

```bash
ansible-playbook -i inventory/mycluster/inventory.ini reset.yml -b
```

## Troubleshooting

### Preflight Failures

```bash
# Check preflight logs
cat ansible/artifacts/run-preflight-rhel10.log

# Re-run preflight
ansible-playbook -i /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml -v
```

### Deployment Failures

```bash
# Check deployment logs
# Logs are verbose - look for FAILED

# Re-run from specific tag
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b --start-at-task="<task name>"
```

### Node Not Joining

```bash
# Check kubelet on node
ssh <node> "systemctl status kubelet"
ssh <node> "journalctl -xeu kubelet"
```

## Related Documentation

- [Cluster Deployment](cluster-deployment.md)
- [Monitoring Deployment](monitoring-deployment.md)
- [Kubespray Integration Summary](../development/ai-agent-implementation.md)
