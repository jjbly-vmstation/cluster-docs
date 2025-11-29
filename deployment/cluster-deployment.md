# Cluster Deployment

Deploy the core Kubernetes cluster using kubeadm.

## Overview

This guide covers deploying the Debian-based Kubernetes cluster using kubeadm. For production deployments, consider [Kubespray](kubespray-deployment.md).

## Prerequisites

- Debian 12 nodes configured
- SSH access to all nodes
- Inventory configured in `ansible/inventory/hosts.yml`

See [Prerequisites](../getting-started/prerequisites.md) for full requirements.

## Deployment Command

```bash
./deploy.sh debian
```

## Deployment Phases

### Phase 0: Install Kubernetes Binaries

Installs kubeadm, kubelet, kubectl on all nodes.

### Phase 1: System Preparation

- Configure sysctl settings
- Disable swap
- Load kernel modules

### Phase 2: CNI Plugins Installation

Installs CNI plugins on all nodes.

### Phase 3: Control Plane Initialization

```bash
kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --service-cidr=10.96.0.0/12 \
  --control-plane-endpoint=192.168.4.63:6443
```

### Phase 4: Worker Node Join

Joins worker nodes using kubeadm join token.

### Phase 5: Flannel CNI Deployment

Deploys Flannel overlay network.

### Phase 6: Wait for Nodes Ready

Waits for all nodes to reach Ready state.

### Phase 7: Deploy Workloads

Deploys monitoring stack and Jellyfin.

## Manual Deployment

### Initialize Control Plane

```bash
# On masternode
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --service-cidr=10.96.0.0/12

# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Deploy CNI

```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

### Join Workers

```bash
# Get join command
kubeadm token create --print-join-command

# On worker nodes
sudo kubeadm join 192.168.4.63:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

## Verification

```bash
# Check nodes
kubectl get nodes
# Expected: All nodes Ready

# Check system pods
kubectl get pods -n kube-system
# Expected: All pods Running
```

## Configuration

### Kubernetes Version

Edit `ansible/inventory/hosts.yml`:

```yaml
all:
  vars:
    kubernetes_version: "1.29"
```

### Network CIDR

```yaml
all:
  vars:
    pod_network_cidr: "10.244.0.0/16"
    service_network_cidr: "10.96.0.0/12"
```

### CNI Plugin

```yaml
all:
  vars:
    cni_plugin: flannel  # or calico
```

## Reset Cluster

```bash
./deploy.sh reset
```

This will:
- Drain all nodes
- Remove Kubernetes state
- Clean CNI artifacts
- Reset iptables rules

## Troubleshooting

### Node Not Joining

```bash
# Check join token
kubeadm token list

# Create new token
kubeadm token create --print-join-command
```

### Pods Not Ready

```bash
# Check pod status
kubectl describe pod <pod-name> -n <namespace>

# Check kubelet
journalctl -xeu kubelet
```

### CNI Issues

```bash
# Check Flannel pods
kubectl get pods -n kube-flannel

# Check CNI config
cat /etc/cni/net.d/10-flannel.conflist
```

## Related Documentation

- [Kubespray Deployment](kubespray-deployment.md)
- [Monitoring Deployment](monitoring-deployment.md)
- [Prerequisites](../getting-started/prerequisites.md)
