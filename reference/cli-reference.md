# CLI Reference

Command line tools and usage.

## deploy.sh

Main deployment script.

### Usage

```bash
./deploy.sh <command> [options]
```

### Commands

| Command | Description |
|---------|-------------|
| `help` | Show help message |
| `debian` | Deploy Debian cluster (kubeadm) |
| `kubespray` | Deploy Kubespray cluster |
| `rke2` | Deploy RKE2 on homelab |
| `monitoring` | Deploy monitoring stack |
| `infrastructure` | Deploy infrastructure services |
| `setup` | Setup auto-sleep monitoring |
| `reset` | Reset cluster to clean state |
| `all` | Deploy everything |

### Options

| Option | Description |
|--------|-------------|
| `--yes` | Skip confirmations |
| `--check` | Dry-run mode |
| `--with-rke2` | Include RKE2 deployment |
| `--log-dir=<path>` | Custom log directory |

### Examples

```bash
# Deploy Debian cluster
./deploy.sh debian

# Deploy with dry-run
./deploy.sh monitoring --check

# Deploy all non-interactively
./deploy.sh all --with-rke2 --yes

# Reset cluster
./deploy.sh reset --yes
```

## Validation Scripts

### validate-monitoring-stack.sh

Validates monitoring stack health.

```bash
./scripts/validate-monitoring-stack.sh
```

**Checks:**
1. Pod status
2. Service endpoints
3. PVC bindings
4. Health endpoints
5. DNS resolution
6. Container restarts
7. Log analysis

### test-complete-validation.sh

Complete validation suite.

```bash
./tests/test-complete-validation.sh
```

**Phases:**
1. Configuration validation
2. Monitoring health
3. Sleep/wake cycle (optional)

## Diagnostic Scripts

### diagnose-monitoring-stack.sh

Comprehensive diagnostics.

```bash
./scripts/diagnose-monitoring-stack.sh
```

**Output:**
- Pod status and logs
- Service configurations
- PVC status
- Events
- Analysis and recommendations

### remediate-monitoring-stack.sh

Automated remediation.

```bash
./scripts/remediate-monitoring-stack.sh
```

**Features:**
- Interactive confirmation
- Backup creation
- Common fixes applied
- Post-fix validation

## Kubespray Scripts

### run-kubespray.sh

Stage Kubespray for deployment.

```bash
./scripts/run-kubespray.sh
```

**Actions:**
- Clone/update Kubespray
- Create virtual environment
- Install requirements
- Create inventory template

## Test Scripts

### test-sleep-wake-cycle.sh

Test auto-sleep and WoL.

```bash
./tests/test-sleep-wake-cycle.sh
```

⚠️ Destructive - requires confirmation.

### test-idempotence.sh

Test deployment idempotency.

```bash
./tests/test-idempotence.sh [cycles]
```

### pre-deployment-checklist.sh

Pre-deployment validation.

```bash
./tests/pre-deployment-checklist.sh
```

## kubectl

Kubernetes command-line tool.

### Configuration

```bash
# Default kubeconfig
export KUBECONFIG=/etc/kubernetes/admin.conf

# Or specify per-command
kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes
```

### Common Commands

```bash
# Nodes
kubectl get nodes
kubectl describe node <name>

# Pods
kubectl get pods -A
kubectl logs <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>

# Services
kubectl get svc -A
kubectl get endpoints -A

# Deployments
kubectl get deployments -A
kubectl rollout restart deployment <name> -n <namespace>
```

## wakeonlan

Wake sleeping nodes.

### Usage

```bash
wakeonlan <mac-address>
```

### Examples

```bash
# Wake storagenodet3500
wakeonlan b8:ac:6f:7e:6c:9d

# Wake homelab
wakeonlan d0:94:66:30:d6:63
```

## Related Documentation

- [Quick Start](../getting-started/quick-start.md)
- [Operations](../operations/README.md)
