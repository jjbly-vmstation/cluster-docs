# Ansible Playbooks

Documentation for VMStation Ansible playbooks.

## Overview

Playbooks are located in `ansible/playbooks/` in the main repository.

## Main Playbooks

### deploy-cluster.yaml

Deploys the Debian Kubernetes cluster.

**Phases:**
1. Install Kubernetes binaries
2. System preparation
3. CNI plugins installation
4. Control plane initialization
5. Worker node join
6. Flannel deployment
7. Wait for nodes Ready
8. Deploy workloads

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/deploy-cluster.yaml
```

### run-preflight-rhel10.yml

Prepares RHEL10 nodes for Kubernetes.

**Tasks:**
- Install Python3
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

### deploy-monitoring.yaml

Deploys the monitoring stack.

**Components:**
- Prometheus
- Grafana
- Loki
- Promtail
- Node Exporter
- Blackbox Exporter

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/deploy-monitoring.yaml
```

### deploy-infrastructure-services.yaml

Deploys infrastructure services.

**Components:**
- NTP/Chrony
- Syslog server
- Kerberos (optional)

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/deploy-infrastructure-services.yaml
```

### install-rke2-homelab.yml

Installs RKE2 on homelab node.

**Tasks:**
- Run preflight checks
- Download RKE2
- Configure server
- Start service
- Deploy monitoring
- Fetch kubeconfig

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/install-rke2-homelab.yml --ask-vault-pass
```

### reset-cluster.yaml

Resets the cluster to clean state.

**Tasks:**
- Drain nodes
- Remove Kubernetes state
- Clean CNI artifacts
- Reset iptables
- Uninstall RKE2

**Usage:**
```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/reset-cluster.yaml
```

## Playbook Variables

### Common Variables

```yaml
kubernetes_version: "1.29"
pod_network_cidr: "10.244.0.0/16"
service_network_cidr: "10.96.0.0/12"
cni_plugin: flannel
```

### Inventory Variables

```yaml
ansible_host: 192.168.4.63
ansible_user: root
ansible_ssh_private_key_file: /root/.ssh/id_k3s
```

## Running Playbooks

### With deploy.sh

```bash
./deploy.sh debian        # Runs deploy-cluster.yaml
./deploy.sh monitoring    # Runs deploy-monitoring.yaml
./deploy.sh infrastructure # Runs deploy-infrastructure-services.yaml
./deploy.sh rke2          # Runs install-rke2-homelab.yml
./deploy.sh reset         # Runs reset-cluster.yaml
```

### Direct Execution

```bash
cd ansible
ansible-playbook -i inventory/hosts.yml playbooks/<playbook>.yaml
```

### Check Mode (Dry Run)

```bash
ansible-playbook -i inventory/hosts.yml playbooks/<playbook>.yaml --check
```

### Verbose Output

```bash
ansible-playbook -i inventory/hosts.yml playbooks/<playbook>.yaml -v
```

## Related Documentation

- [Roles](roles.md)
- [Best Practices](best-practices.md)
- [Deployment](../../deployment/README.md)
