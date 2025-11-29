# CLI Reference

Command-line interface reference for VMStation.

## deploy.sh

Main deployment script.

### Usage

```bash
./deploy.sh <command> [options]
```

### Commands

| Command | Description |
|---------|-------------|
| `debian` | Deploy Kubernetes on Debian nodes |
| `rke2` | Deploy RKE2 on homelab |
| `kubespray` | Deploy with Kubespray |
| `monitoring` | Deploy monitoring stack |
| `infrastructure` | Deploy infrastructure services |
| `setup` | Configure auto-sleep |
| `reset` | Reset all clusters |
| `all` | Deploy everything |
| `help` | Show help |

### Options

| Option | Description |
|--------|-------------|
| `--yes` | Skip confirmations |
| `--check` | Dry-run mode |
| `--with-rke2` | Include RKE2 deployment |
| `--log-dir <dir>` | Custom log directory |

### Examples

```bash
# Deploy Debian cluster
./deploy.sh debian

# Deploy with confirmation skip
./deploy.sh debian --yes

# Dry run
./deploy.sh monitoring --check

# Full deployment
./deploy.sh all --with-rke2 --yes

# Reset everything
./deploy.sh reset --yes
```

## kubectl

Kubernetes command-line tool.

### Configuration

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### Common Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Pods
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>

# Services
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>

# Deployments
kubectl get deployments -n <namespace>
kubectl scale deployment <name> --replicas=3 -n <namespace>
kubectl rollout restart deployment/<name> -n <namespace>
```

## Ansible

### Run Playbook

```bash
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/<playbook>.yaml
```

### Options

| Option | Description |
|--------|-------------|
| `--check` | Dry run |
| `-v/-vv/-vvv` | Verbosity |
| `--limit <host>` | Limit to hosts |
| `--ask-vault-pass` | Prompt for vault password |

## Scripts

### Diagnostics

```bash
# Monitoring diagnostics
./scripts/diagnose-monitoring-stack.sh

# Monitoring validation
./scripts/validate-monitoring-stack.sh

# Remediation
./scripts/remediate-monitoring-stack.sh
```

### Testing

```bash
# Complete validation
./tests/test-complete-validation.sh

# Time sync validation
./tests/validate-time-sync.sh

# Sleep/wake test
./tests/test-sleep-wake-cycle.sh
```

### Kubespray

```bash
# Stage Kubespray
./scripts/run-kubespray.sh
```

## Wake-on-LAN

```bash
# Install
apt install wakeonlan

# Wake node
wakeonlan <MAC>

# Wake with broadcast
wakeonlan -i 192.168.4.255 <MAC>
```

## Related Documentation

- [Cluster Management](../operations/cluster-management.md)
- [Ansible Playbooks](../components/ansible/playbooks.md)
