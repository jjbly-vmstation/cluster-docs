# Ansible Playbooks

This document covers the Ansible playbooks used in VMStation.

## Playbook Overview

### Deployment Playbooks

| Playbook | Purpose | Target |
|----------|---------|--------|
| deploy-cluster.yaml | Deploy Kubernetes | Debian nodes |
| install-rke2-homelab.yml | Deploy RKE2 | RHEL10 nodes |
| run-preflight-rhel10.yml | Prepare RHEL10 | homelab |

### Operational Playbooks

| Playbook | Purpose |
|----------|---------|
| reset-cluster.yaml | Reset cluster |
| fix-loki-config.yaml | Fix Loki issues |
| setup-autosleep.yaml | Configure auto-sleep |

## Deployment Playbooks

### deploy-cluster.yaml

Deploys Kubernetes cluster on Debian nodes using kubeadm.

**Phases:**
1. Phase 0: Install Kubernetes binaries
2. Phase 1: System preparation
3. Phase 2: CNI plugins installation
4. Phase 3: Control plane initialization
5. Phase 4: Worker node join
6. Phase 5: Flannel CNI deployment
7. Phase 6: Cluster validation
8. Phase 7: Application deployment

**Usage:**
```bash
./deploy.sh debian
# OR
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/deploy-cluster.yaml
```

**Idempotency:** ✅ Safe to run multiple times

### install-rke2-homelab.yml

Deploys RKE2 on RHEL10 homelab node.

**Features:**
- Downloads and installs RKE2
- Configures Flannel CNI
- Enables RKE2 server service
- Fetches kubeconfig

**Usage:**
```bash
./deploy.sh rke2
# OR
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/install-rke2-homelab.yml
```

### run-preflight-rhel10.yml

Prepares RHEL10 nodes for Kubernetes deployment.

**Actions:**
- Install Python3 packages
- Configure chrony
- Setup sudoers
- Open firewall ports
- Configure SELinux
- Load kernel modules
- Apply sysctl settings
- Disable swap

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/run-preflight-rhel10.yml
```

## Cleanup Playbooks

### reset-cluster.yaml

Comprehensive cluster reset.

**Actions:**
- Run kubeadm reset
- Stop services
- Remove directories
- Clean network interfaces
- Flush iptables

**Usage:**
```bash
./deploy.sh reset
# OR
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/reset-cluster.yaml
```

### uninstall-rke2-homelab.yml

Removes RKE2 from homelab node.

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/uninstall-rke2-homelab.yml
```

## Operational Playbooks

### fix-loki-config.yaml

Fixes Loki configuration and permissions.

**Actions:**
- Reapply Loki manifest
- Fix directory permissions
- Restart Loki
- Validate readiness

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/fix-loki-config.yaml
```

### setup-autosleep.yaml

Configures automatic cluster sleep.

**Features:**
- Creates monitoring script
- Installs systemd timer
- Configures WoL handlers

**Usage:**
```bash
./deploy.sh setup
# OR
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/setup-autosleep.yaml
```

## Running Playbooks

### Basic Execution

```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/<playbook>.yaml
```

### With Vault Password

```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/<playbook>.yaml --ask-vault-pass
```

### Dry Run

```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/<playbook>.yaml --check
```

### Verbose Output

```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/<playbook>.yaml -vvv
```

### Limit to Specific Hosts

```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/<playbook>.yaml --limit homelab
```

## Error Handling

All playbooks implement error handling:

- Detailed diagnostics on failure
- Logs saved to `ansible/artifacts/`
- Clean exit on critical failures

## Related Documentation

- [Ansible Roles](roles.md)
- [Best Practices](best-practices.md)
- [Cluster Deployment](../../deployment/cluster-deployment.md)
