# Upgrades

Upgrading VMStation components.

## Kubernetes Upgrade

### Pre-Upgrade Checklist

- [ ] Backup etcd
- [ ] Backup monitoring data
- [ ] Check upgrade path (max 1 minor version)
- [ ] Review release notes
- [ ] Test in staging if available

### Using Kubespray

```bash
cd .cache/kubespray
source .venv/bin/activate

# Update version in group_vars
# Edit k8s-cluster.yml
kube_version: v1.30.0

# Run upgrade
ansible-playbook -i inventory/mycluster/inventory.ini upgrade-cluster.yml -b
```

### Using kubeadm

#### Upgrade Control Plane

```bash
# Check available versions
apt update
apt-cache madison kubeadm

# Upgrade kubeadm
apt install -y kubeadm=1.30.0-*

# Check upgrade plan
kubeadm upgrade plan

# Apply upgrade
kubeadm upgrade apply v1.30.0

# Upgrade kubelet and kubectl
apt install -y kubelet=1.30.0-* kubectl=1.30.0-*
systemctl daemon-reload
systemctl restart kubelet
```

#### Upgrade Worker Nodes

```bash
# On control plane - drain node
kubectl drain <nodename> --ignore-daemonsets --delete-emptydir-data

# On worker node
apt update
apt install -y kubeadm=1.30.0-* kubelet=1.30.0-* kubectl=1.30.0-*
kubeadm upgrade node
systemctl daemon-reload
systemctl restart kubelet

# On control plane - uncordon
kubectl uncordon <nodename>
```

### Verify Upgrade

```bash
kubectl get nodes
kubectl version
```

## Monitoring Stack Upgrade

### Prometheus

```bash
# Update image in manifest
kubectl set image statefulset/prometheus prometheus=prom/prometheus:v2.50.0 -n monitoring

# Or edit manifest and apply
kubectl apply -f manifests/monitoring/prometheus.yaml
```

### Grafana

```bash
kubectl set image deployment/grafana grafana=grafana/grafana:10.3.0 -n monitoring
```

### Loki

```bash
kubectl set image statefulset/loki loki=grafana/loki:2.9.0 -n monitoring
```

## Application Upgrades

### Rolling Update

```bash
kubectl set image deployment/<name> <container>=<image>:<tag> -n <namespace>
```

### Blue-Green Deployment

1. Deploy new version with different name
2. Test new version
3. Switch service selector
4. Delete old version

### Canary Deployment

1. Deploy small replica set of new version
2. Monitor for errors
3. Gradually shift traffic
4. Complete rollout or rollback

## Rollback Procedures

### Kubernetes Rollback

```bash
# kubeadm doesn't support rollback
# Restore from etcd backup
```

### Deployment Rollback

```bash
kubectl rollout undo deployment/<name> -n <namespace>
kubectl rollout undo deployment/<name> --to-revision=2 -n <namespace>
```

### View Rollout History

```bash
kubectl rollout history deployment/<name> -n <namespace>
```

## OS Updates

### Debian Nodes

```bash
# On each node
apt update
apt upgrade -y
apt autoremove -y

# Reboot if kernel updated
reboot
```

### Rolling Node Updates

```bash
# For each node
kubectl drain <nodename> --ignore-daemonsets --delete-emptydir-data
ssh <nodename> "apt update && apt upgrade -y && reboot"
# Wait for node to come back
kubectl uncordon <nodename>
```

## Certificate Renewal

### Check Expiry

```bash
kubeadm certs check-expiration
```

### Renew Certificates

```bash
kubeadm certs renew all
systemctl restart kubelet
```

## Upgrade Timeline

### Recommended Schedule

| Component | Frequency | Notes |
|-----------|-----------|-------|
| Security patches | Immediate | Critical vulnerabilities |
| Minor K8s | Quarterly | e.g., 1.29 → 1.30 |
| Major K8s | Annual | Plan carefully |
| Monitoring | As needed | When features needed |
| OS | Monthly | Security updates |

## Related Documentation

- [Cluster Management](cluster-management.md)
- [Backup & Restore](backup-restore.md)
- [Troubleshooting](../troubleshooting/README.md)
