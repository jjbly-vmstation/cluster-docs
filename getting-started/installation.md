# Installation Guide

Detailed installation instructions for VMStation.

## Clone Repository

```bash
git clone https://github.com/JashandeepJustinBains/VMStation.git
cd VMStation
```

## Deployment Options

VMStation supports multiple deployment paths:

| Option | Description | Use Case |
|--------|-------------|----------|
| Kubespray | Production-grade Kubernetes | Recommended |
| Debian kubeadm | Legacy kubeadm deployment | Debian-only |
| RKE2 | Rancher Kubernetes Engine | RHEL nodes |

## Option 1: Kubespray Deployment (Recommended)

### Prepare RHEL10 Node

Run preflight checks on RHEL10 nodes:

```bash
ansible-playbook -i ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml
```

This will:
- Install Python3 and required packages
- Configure chrony (time sync)
- Setup sudoers for ansible user
- Open firewall ports
- Configure SELinux
- Load kernel modules
- Apply sysctl settings
- Disable swap

### Stage Kubespray

```bash
./scripts/run-kubespray.sh
```

This will:
- Clone Kubespray into `.cache/kubespray`
- Create Python virtual environment
- Install Kubespray requirements
- Create inventory template

### Customize Inventory

Edit `.cache/kubespray/inventory/mycluster/inventory.ini`:

```ini
[all]
homelab ansible_host=192.168.4.62 ansible_user=jashandeepjustinbains

[kube_control_plane]
homelab

[kube_node]
homelab

[etcd]
homelab

[k8s_cluster:children]
kube_control_plane
kube_node
```

### Deploy Cluster

```bash
cd .cache/kubespray
source .venv/bin/activate
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml
```

### Access Cluster

```bash
scp jashandeepjustinbains@192.168.4.62:.kube/config ~/.kube/config-homelab
export KUBECONFIG=~/.kube/config-homelab
kubectl get nodes
```

## Option 2: Debian Cluster (kubeadm)

### Deploy Cluster

```bash
./deploy.sh debian
```

This runs:
- Phase 0: Install Kubernetes binaries
- Phase 1: System preparation
- Phase 2: CNI plugins installation
- Phase 3: Control plane initialization
- Phase 4: Worker node join
- Phase 5: Flannel CNI deployment
- Phase 6: Wait for nodes Ready
- Phase 7: Deploy monitoring and Jellyfin

### Deploy Monitoring

```bash
./deploy.sh monitoring
```

Components deployed:
- Prometheus (metrics collection)
- Grafana (dashboards)
- Loki (log aggregation)
- Promtail (log shipping)
- Node Exporter (system metrics)
- Blackbox Exporter (probes)

### Deploy Infrastructure

```bash
./deploy.sh infrastructure
```

Components deployed:
- NTP/Chrony DaemonSet
- Syslog Server
- Kerberos/FreeIPA (optional)

### Setup Auto-Sleep

```bash
./deploy.sh setup
```

## Option 3: RKE2 Deployment

### Deploy RKE2

```bash
./deploy.sh rke2
```

This will:
- Run preflight checks
- Download and install RKE2
- Configure and start RKE2 server
- Deploy monitoring components
- Fetch kubeconfig

### Access RKE2 Cluster

```bash
export KUBECONFIG=ansible/artifacts/homelab-rke2-kubeconfig.yaml
kubectl get nodes
```

## All-in-One Deployment

Deploy everything in one command:

```bash
./deploy.sh all --with-rke2 --yes
```

Then deploy monitoring and infrastructure:

```bash
./deploy.sh monitoring
./deploy.sh infrastructure
./deploy.sh setup
```

## Storage Class Setup

After cluster deployment, install storage class:

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.24/deploy/local-path-storage.yaml

kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Verify Installation

```bash
# Check nodes
kubectl get nodes

# Check all pods
kubectl get pods -A

# Check monitoring
kubectl get pods -n monitoring

# Run validation
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Next Steps

- [First Deployment](first-deployment.md) - Deploy your first workload
- [Cluster Management](../operations/cluster-management.md) - Manage the cluster
- [Troubleshooting](../troubleshooting/README.md) - Common issues
