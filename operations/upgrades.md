# Upgrades

This guide covers upgrading VMStation components.

## Upgrade Overview

Components to upgrade:
- Kubernetes version
- Monitoring stack (Prometheus, Grafana, Loki)
- Infrastructure services
- Container images

## Kubernetes Upgrade

### Check Current Version

```bash
kubectl get nodes
kubectl version --short
```

### Using Kubespray

#### Update Kubespray

```bash
cd .cache/kubespray
git fetch --tags
git checkout v2.24.0  # Latest version
```

#### Update Cluster Variables

Edit `inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml`:

```yaml
kube_version: v1.30.0
```

#### Run Upgrade

```bash
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini upgrade-cluster.yml -b
```

#### Upgrade Specific Node

```bash
ansible-playbook -i inventory/mycluster/inventory.ini upgrade-cluster.yml -b --limit homelab
```

### Using kubeadm (Legacy)

#### Control Plane

```bash
# On masternode
sudo apt update
sudo apt install -y kubeadm=1.30.0-1.1

# Check upgrade plan
sudo kubeadm upgrade plan

# Apply upgrade
sudo kubeadm upgrade apply v1.30.0

# Upgrade kubelet
sudo apt install -y kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

#### Worker Nodes

```bash
# On each worker
sudo apt update
sudo apt install -y kubeadm=1.30.0-1.1
sudo kubeadm upgrade node

sudo apt install -y kubelet=1.30.0-1.1 kubectl=1.30.0-1.1
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Verification

```bash
kubectl get nodes
# All nodes should show new version
```

## Monitoring Stack Upgrade

### Update Manifests

```bash
cd /srv/monitoring_data/VMStation
git pull
```

### Update Container Images

Edit manifests to update image tags:

```yaml
# manifests/monitoring/prometheus.yaml
image: prom/prometheus:v2.50.0

# manifests/monitoring/grafana.yaml
image: grafana/grafana:10.3.0

# manifests/monitoring/loki.yaml
image: grafana/loki:2.9.0
```

### Apply Updates

```bash
kubectl apply -f manifests/monitoring/
```

### Rolling Restart

```bash
kubectl rollout restart deployment/grafana -n monitoring
kubectl rollout restart statefulset/prometheus -n monitoring
kubectl rollout restart statefulset/loki -n monitoring
kubectl rollout restart daemonset/promtail -n monitoring
kubectl rollout restart daemonset/node-exporter -n monitoring
```

### Verify Upgrade

```bash
# Check pod images
kubectl get pods -n monitoring -o jsonpath='{.items[*].spec.containers[*].image}' | tr ' ' '\n'

# Check pod status
kubectl get pods -n monitoring

# Validate stack
./scripts/validate-monitoring-stack.sh
```

## Infrastructure Services Upgrade

### Update Chrony

```bash
kubectl apply -f manifests/infrastructure/chrony.yaml
kubectl rollout restart daemonset/chrony-ntp -n infrastructure
```

## Helm Chart Upgrades (If Using Helm)

### Update Repositories

```bash
helm repo update
```

### Upgrade Release

```bash
helm upgrade <release> <chart> -n <namespace>
```

## Rollback Procedures

### Kubernetes Rollback

If upgrade fails:

```bash
# Kubespray doesn't have rollback - restore from backup or reinstall
```

### Application Rollback

```bash
kubectl rollout undo deployment/<name> -n <namespace>
kubectl rollout undo statefulset/<name> -n <namespace>
```

### View Rollout History

```bash
kubectl rollout history deployment/<name> -n <namespace>
```

## Pre-Upgrade Checklist

- [ ] Backup etcd
- [ ] Backup monitoring data
- [ ] Review release notes
- [ ] Test in non-production (if available)
- [ ] Document current versions
- [ ] Notify users of maintenance window

## Post-Upgrade Validation

```bash
# Check cluster health
kubectl get nodes
kubectl get pods -A | grep -v Running

# Validate monitoring
./scripts/validate-monitoring-stack.sh

# Run full test suite
./tests/test-complete-validation.sh
```

## Version Compatibility

### Supported Kubernetes Versions

| Kubespray | Kubernetes |
|-----------|------------|
| v2.24.0 | v1.28.x - v1.29.x |
| v2.23.0 | v1.27.x - v1.28.x |

### Monitoring Stack Compatibility

| Component | Tested Version |
|-----------|---------------|
| Prometheus | v2.48.x - v2.50.x |
| Grafana | v10.0.x - v10.3.x |
| Loki | v2.8.x - v2.9.x |

## Related Documentation

- [Cluster Management](cluster-management.md)
- [Backup & Restore](backup-restore.md)
- [Troubleshooting](../troubleshooting/common-issues.md)
