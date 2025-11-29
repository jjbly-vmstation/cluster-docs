# Kubespray Deployment

Kubespray is the recommended method for deploying production-grade Kubernetes clusters on VMStation.

## Overview

Kubespray provides:
- Standard upstream Kubernetes (like kubeadm)
- Multiple CNI options (Calico, Flannel, Weave, Cilium)
- Multi-node cluster support
- Production-grade automation
- Active community support

## Prerequisites

### RHEL10 Node Requirements

- Python 3 installed
- Chrony configured
- Firewall ports open
- SELinux configured
- Kernel modules loaded
- Swap disabled

## Quick Start

### Step 1: Preflight Checks

```bash
ansible-playbook -i ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml
```

This configures:
- Python3 packages
- Chrony time sync
- Sudoers configuration
- Firewall ports
- SELinux settings
- Kernel modules
- Sysctl parameters
- Swap disablement

### Step 2: Stage Kubespray

```bash
./scripts/run-kubespray.sh
```

This:
- Clones Kubespray to `.cache/kubespray`
- Creates Python virtual environment
- Installs Kubespray requirements
- Creates inventory template

### Step 3: Customize Inventory

Edit `.cache/kubespray/inventory/mycluster/inventory.ini`:

```ini
[all]
masternode ansible_host=192.168.4.63
storagenodet3500 ansible_host=192.168.4.61
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

### Step 4: Customize Cluster Variables

Edit `.cache/kubespray/inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml`:

```yaml
kube_version: v1.29.6
kube_network_plugin: calico  # or flannel
container_manager: containerd
kube_proxy_mode: ipvs
```

### Step 5: Deploy Cluster

```bash
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b
```

## Full Deployment with VMStation

Use the integrated deployment:

```bash
./deploy.sh kubespray
```

This runs:
1. Preflight checks
2. Kubespray staging
3. Cluster deployment
4. Monitoring deployment
5. Infrastructure deployment

## Kubespray Configuration

### CNI Options

```yaml
# Calico (default)
kube_network_plugin: calico

# Flannel
kube_network_plugin: flannel

# Cilium
kube_network_plugin: cilium
```

### Container Runtime

```yaml
# containerd (default)
container_manager: containerd

# CRI-O
container_manager: crio

# Docker (deprecated)
container_manager: docker
```

### Network Settings

```yaml
# Pod network CIDR
kube_pods_subnet: 10.244.0.0/16

# Service network CIDR
kube_service_addresses: 10.96.0.0/12

# DNS service IP
dns_service_ip: 10.233.0.3
```

## Access Cluster

### On Control Plane

```bash
export KUBECONFIG=/root/.kube/config
kubectl get nodes
```

### From Remote

```bash
scp root@192.168.4.63:.kube/config ~/.kube/config-vmstation
export KUBECONFIG=~/.kube/config-vmstation
kubectl get nodes
```

## Upgrade Cluster

### Upgrade Kubernetes Version

Edit `k8s-cluster.yml`:
```yaml
kube_version: v1.30.0
```

Run upgrade playbook:
```bash
ansible-playbook -i inventory/mycluster/inventory.ini upgrade-cluster.yml -b
```

### Upgrade Specific Node

```bash
ansible-playbook -i inventory/mycluster/inventory.ini upgrade-cluster.yml -b --limit homelab
```

## Add Nodes

### Add to Inventory

```ini
[kube_node]
storagenodet3500
homelab
newnode ansible_host=192.168.4.64
```

### Run Scale Playbook

```bash
ansible-playbook -i inventory/mycluster/inventory.ini scale.yml -b
```

## Remove Nodes

### Drain and Delete

```bash
kubectl drain newnode --ignore-daemonsets --delete-emptydir-data
kubectl delete node newnode
```

### Or use remove-node playbook

```bash
ansible-playbook -i inventory/mycluster/inventory.ini remove-node.yml -b --extra-vars "node=newnode"
```

## Reset Cluster

### Full Reset

```bash
ansible-playbook -i inventory/mycluster/inventory.ini reset.yml -b
```

### Or use VMStation reset

```bash
./deploy.sh reset
```

## Comparison: Kubespray vs RKE2

| Feature | Kubespray | RKE2 |
|---------|-----------|------|
| Kubernetes Type | Upstream | Rancher K8s |
| Installation | Ansible | Single binary |
| CNI | Multiple options | Canal only |
| Complexity | Medium | Low |
| Flexibility | High | Medium |
| Updates | Standard upgrade | RKE2 upgrade |
| RHEL Support | Via preflight | Native |

**Choose Kubespray when:**
- Need specific CNI
- Want standard Kubernetes
- Multi-node clusters
- Advanced customization

**Choose RKE2 when:**
- Simple single-node
- Want "batteries included"
- Prefer minimal configuration

## Troubleshooting

### Preflight Failures

```bash
# Check preflight role
ansible-playbook -i ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml -vvv
```

### Kubespray Deployment Failures

```bash
# Run with verbose
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml -b -vvv

# Check specific host
ansible-playbook ... --limit homelab
```

### Common Issues

**Python not found:**
```bash
# Install on RHEL
sudo dnf install python3
```

**Firewall blocking:**
```bash
# Check firewall
sudo firewall-cmd --list-all
```

**SELinux denials:**
```bash
# Check audit log
sudo ausearch -m AVC -ts recent
```

## Related Documentation

- [Cluster Deployment](cluster-deployment.md)
- [Preflight Role](../components/ansible/roles.md)
- [Architecture Overview](../architecture/overview.md)
