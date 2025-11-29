# Prerequisites

Before deploying VMStation, ensure you have the following requirements met.

## Hardware Requirements

### Nodes

| Node | Role | CPU | RAM | Disk | Notes |
|------|------|-----|-----|------|-------|
| masternode | Control Plane | 4 cores | 8GB | 100GB | Always-on, hosts monitoring |
| storagenodet3500 | Worker, Storage | 4 cores | 8GB | 500GB+ | Auto-sleep enabled |
| homelab | Compute | 4 cores | 8GB | 100GB | RHEL10, auto-sleep enabled |

### Network

- All nodes on same subnet (192.168.4.0/24)
- Wake-on-LAN enabled in BIOS for worker nodes
- Static IP addresses configured
- SSH access from control node to all nodes

### Wake-on-LAN MAC Addresses

| Node | MAC Address |
|------|-------------|
| storagenodet3500 | b8:ac:6f:7e:6c:9d |
| homelab | d0:94:66:30:d6:63 |

## Software Requirements

### Operating Systems

| Node | OS |
|------|-----|
| masternode | Debian 12 (Bookworm) |
| storagenodet3500 | Debian 12 (Bookworm) |
| homelab | RHEL 10 or compatible |

### Control Machine (masternode)

- **Ansible**: 2.9+ (recommended: 2.14+)
- **Python**: 3.8+
- **SSH**: OpenSSH client
- **Git**: For cloning repositories

### Installation Commands

**Debian/Ubuntu:**

```bash
# Install Ansible
sudo apt update
sudo apt install -y ansible python3-pip git

# Verify versions
ansible --version
python3 --version
```

**RHEL/Fedora:**

```bash
# Install Ansible
sudo dnf install -y ansible python3-pip git

# Verify versions
ansible --version
python3 --version
```

## SSH Configuration

### Generate SSH Keys

```bash
# Generate key pair (if not exists)
ssh-keygen -t ed25519 -f ~/.ssh/id_k3s -N ""

# Copy to all nodes
ssh-copy-id -i ~/.ssh/id_k3s root@192.168.4.61  # storagenodet3500
ssh-copy-id -i ~/.ssh/id_k3s user@192.168.4.62  # homelab
```

### Verify Connectivity

```bash
# Test SSH access
ssh -i ~/.ssh/id_k3s root@192.168.4.61 "hostname"
ssh -i ~/.ssh/id_k3s user@192.168.4.62 "hostname"
```

### Important: Use Absolute Paths

In Ansible inventory, always use absolute paths for SSH keys:

```yaml
# Correct
ansible_ssh_private_key_file: /root/.ssh/id_k3s

# Wrong - will cause issues
ansible_ssh_private_key_file: ~/.ssh/id_k3s
```

## Network Requirements

### Ports

The following ports must be open between nodes:

| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | TCP | SSH |
| 6443 | TCP | Kubernetes API |
| 2379-2380 | TCP | etcd |
| 10250 | TCP | Kubelet API |
| 10251 | TCP | kube-scheduler |
| 10252 | TCP | kube-controller-manager |
| 30000-32767 | TCP | NodePort Services |

### Firewall Configuration

**Debian:**

```bash
# Usually no firewall by default, but if using ufw:
sudo ufw allow 6443/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
```

**RHEL:**

```bash
# Open Kubernetes ports
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=30000-32767/tcp
sudo firewall-cmd --reload
```

## Storage Requirements

### Local Storage Paths

The following directories will be created on masternode:

| Path | Purpose | Size |
|------|---------|------|
| /srv/monitoring_data/ | Monitoring data | 50GB+ |
| /srv/monitoring_data/prometheus/ | Prometheus TSDB | 20GB |
| /srv/monitoring_data/loki/ | Loki logs | 20GB |
| /srv/monitoring_data/grafana/ | Grafana data | 5GB |

### Create Storage Directories

```bash
# On masternode
sudo mkdir -p /srv/monitoring_data/{prometheus,loki,grafana}
sudo chmod 755 /srv/monitoring_data
```

## Time Synchronization

All nodes must have synchronized time. Chrony is deployed automatically, but ensure:

```bash
# Check time sync status
timedatectl status

# If using systemd-timesyncd, it will be replaced with chrony
```

## Vault Configuration (Optional)

For RHEL nodes requiring sudo passwords:

```bash
# Create vault file
ansible-vault create ansible/inventory/group_vars/secrets.yml
```

Add your sudo password:

```yaml
vault_homelab_sudo_password: "your_sudo_password"
```

## Pre-Flight Checklist

Before starting deployment, verify:

- [ ] All nodes powered on and accessible
- [ ] SSH keys configured and tested
- [ ] Network connectivity between all nodes
- [ ] Required ports open
- [ ] Storage directories created
- [ ] Time synchronized (within 5 seconds)
- [ ] Ansible installed on control node

### Automated Pre-Flight Check

```bash
# Run pre-deployment checklist
./tests/pre-deployment-checklist.sh
```

## Next Steps

Once prerequisites are met, proceed to:

- [Quick Start](quick-start.md) - Fast deployment
- [Installation](installation.md) - Detailed installation
