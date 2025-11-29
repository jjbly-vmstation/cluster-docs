# Prerequisites

Requirements for deploying VMStation.

## Hardware Requirements

### Node Specifications

| Node | CPU | RAM | Disk | Notes |
|------|-----|-----|------|-------|
| masternode | 4 cores | 8GB | 100GB | Always-on, control plane |
| storagenodet3500 | 4 cores | 8GB | 500GB+ | Storage workloads |
| homelab | 4 cores | 8GB | 100GB | Compute workloads |

### Network Requirements

- All nodes on same subnet (192.168.4.0/24)
- Wake-on-LAN enabled in BIOS
- Static IP addresses configured
- SSH access between nodes

## Software Requirements

### Operating Systems

| Node | OS | Notes |
|------|----|-------|
| masternode | Debian 12 (Bookworm) | Primary deployment target |
| storagenodet3500 | Debian 12 (Bookworm) | Primary deployment target |
| homelab | RHEL 10 | Requires preflight role |

### Required Tools

**On control machine (masternode):**

- **Ansible** 2.9+
- **Python** 3.8+
- **SSH client** with key-based authentication
- **kubectl** (installed during deployment)
- **wakeonlan** for power management

### Install Ansible

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install ansible python3-pip

# Verify
ansible --version
```

## Network Configuration

### IP Addresses

| Node | IP Address | MAC Address |
|------|------------|-------------|
| masternode | 192.168.4.63 | - |
| storagenodet3500 | 192.168.4.61 | b8:ac:6f:7e:6c:9d |
| homelab | 192.168.4.62 | d0:94:66:30:d6:63 |

### Required Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | TCP | SSH |
| 6443 | TCP | Kubernetes API |
| 2379-2380 | TCP | etcd |
| 10250-10252 | TCP | Kubelet, controller, scheduler |
| 30000-32767 | TCP | NodePort services |

## SSH Configuration

### Generate SSH Key

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_k3s
```

### Copy to Nodes

```bash
ssh-copy-id -i ~/.ssh/id_k3s root@192.168.4.61
ssh-copy-id -i ~/.ssh/id_k3s root@192.168.4.62
```

### Test Connectivity

```bash
ssh -i ~/.ssh/id_k3s root@192.168.4.61 hostname
ssh -i ~/.ssh/id_k3s root@192.168.4.62 hostname
```

## Inventory Configuration

Edit `ansible/inventory/hosts.yml`:

```yaml
all:
  vars:
    kubernetes_version: "1.29"
    pod_network_cidr: "10.244.0.0/16"
    service_network_cidr: "10.96.0.0/12"

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

compute_nodes:
  hosts:
    homelab:
      ansible_host: 192.168.4.62
      ansible_user: jashandeepjustinbains
      ansible_ssh_private_key_file: /root/.ssh/id_k3s
```

## Vault Configuration (RHEL)

For RHEL nodes requiring sudo password:

```bash
# Create vault file
ansible-vault create ansible/inventory/group_vars/secrets.yml

# Add content
vault_homelab_sudo_password: "your_sudo_password"
```

## Pre-Deployment Checks

Run the pre-deployment checklist:

```bash
./tests/pre-deployment-checklist.sh
```

This validates:
- SSH connectivity
- Time synchronization
- Required binaries
- Network connectivity

## Wake-on-LAN Setup

### Enable in BIOS

1. Enter BIOS setup
2. Find Power Management settings
3. Enable "Wake on LAN" or "Wake on PCI"
4. Save and exit

### Test WoL

```bash
# Install wakeonlan
sudo apt install wakeonlan

# Test wake
wakeonlan b8:ac:6f:7e:6c:9d  # storagenodet3500
wakeonlan d0:94:66:30:d6:63  # homelab
```

## Next Steps

- [Installation Guide](installation.md) - Begin installation
- [Quick Start](quick-start.md) - Fast deployment
- [Architecture Overview](../architecture/overview.md) - Understand the system
