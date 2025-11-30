# Calico CNI Configuration Documentation

**Date**: November 30, 2025  
**CNI**: Calico  
**Deployment Method**: Kubespray  
**Version**: Latest (managed by Kubespray)

---

## Table of Contents

1. [Overview](#overview)
2. [Calico Architecture](#calico-architecture)
3. [Kubespray Integration](#kubespray-integration)
4. [Network Configuration](#network-configuration)
5. [Network Policies](#network-policies)
6. [Troubleshooting](#troubleshooting)
7. [Validation](#validation)

---

## Overview

The VMStation Kubernetes cluster uses **Calico** as the Container Network Interface (CNI) plugin. Calico is deployed and managed through **Kubespray**, which handles the installation, configuration, and lifecycle management of the CNI.

### Why Calico?

- **High Performance**: Direct Layer 3 routing with BGP
- **Network Policies**: Native support for Kubernetes NetworkPolicy
- **Scalability**: Proven at massive scale (thousands of nodes)
- **IPAM**: Integrated IP address management
- **Observability**: Built-in monitoring and diagnostics
- **Kubespray Native**: First-class support in Kubespray

---

## Calico Architecture

### Components

```
┌─────────────────────────────────────────────────────────┐
│              Calico CNI Architecture                    │
└─────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│                     Control Plane                        │
│                                                          │
│  ┌────────────────┐         ┌──────────────────┐       │
│  │ calico-kube-   │◄───────►│ Kubernetes API   │       │
│  │ controllers    │         │ Server           │       │
│  └────────────────┘         └──────────────────┘       │
└──────────────────────────────────────────────────────────┘
                        │
                        │ Watches Resources
                        │
┌───────────────────────┼───────────────────────────────────┐
│                       ↓             Data Plane            │
│                                                           │
│  ┌───────────────────────────────────────────────────┐  │
│  │              calico-node DaemonSet                │  │
│  │                                                   │  │
│  │  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │  │
│  │  │  Felix   │  │   BIRD   │  │  confd        │  │  │
│  │  │          │  │  (BGP)   │  │  (Config)     │  │  │
│  │  │ Policy   │  │  Routing │  │  Management   │  │  │
│  │  │ Agent    │  │  Daemon  │  │               │  │  │
│  │  └──────────┘  └──────────┘  └───────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                           │
│  ┌───────────────────────────────────────────────────┐  │
│  │           CNI Plugin (calico-cni)                 │  │
│  │  • Invoked by kubelet for pod creation            │  │
│  │  • Allocates IP addresses                         │  │
│  │  • Sets up network interfaces                     │  │
│  └───────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────┘
```

### Calico Components on VMStation Cluster

1. **calico-kube-controllers** (masternode)
   - Watches Kubernetes API for network policy changes
   - Synchronizes Calico configuration with Kubernetes state
   - Manages Calico CRDs (Custom Resource Definitions)

2. **calico-node DaemonSet** (all nodes)
   - **Felix**: Policy agent that programs iptables rules
   - **BIRD**: BGP daemon for route distribution
   - **confd**: Configuration management daemon

3. **calico-cni Plugin**
   - Invoked by kubelet during pod creation/deletion
   - Allocates IP addresses from configured pools
   - Creates veth pairs and routes

---

## Kubespray Integration

### How Calico is Deployed

Calico is automatically deployed by Kubespray during the cluster deployment process. The configuration is managed through Kubespray inventory files.

### Kubespray Deployment Process

```bash
# 1. Kubespray reads inventory configuration
# Location: cluster-infra/inventory/cluster.yml

# 2. Kubespray applies Calico manifests
# Generated manifests based on inventory settings

# 3. Calico components deployed
# - calico-kube-controllers (Deployment)
# - calico-node (DaemonSet)
# - CNI configuration files
```

### Default Configuration (Kubespray)

Kubespray deploys Calico with the following default settings:

```yaml
# Network Plugin
kube_network_plugin: calico

# Calico Version (managed by Kubespray)
calico_version: "v3.27.0"  # Example - actual version from Kubespray

# IP-in-IP Encapsulation
calico_ipip_mode: 'Always'  # or 'CrossSubnet' or 'Never'

# BGP Configuration
calico_bgp_peers: []  # Empty by default (no external BGP peers)

# Network Pools
calico_pool_cidr: 10.244.0.0/16  # Default pod CIDR
calico_pool_blocksize: 26  # /26 blocks (64 IPs per node)

# MTU Configuration
calico_veth_mtu: 1440  # Accounts for IP-in-IP overhead

# Network Backend
calico_datastore: "kubernetes"  # Use Kubernetes API as datastore
```

### Customizing Calico via Kubespray

To customize Calico configuration, edit Kubespray inventory:

**Location**: `cluster-infra/inventory/mycluster/group_vars/k8s_cluster/k8s-net-calico.yml`

```yaml
# Example Calico customizations

# Disable IP-in-IP encapsulation (better performance, same L2 network)
calico_ipip_mode: 'Never'

# Use VXLAN instead of IP-in-IP
calico_vxlan_mode: 'Always'

# Enable Prometheus metrics
calico_felix_prometheusmetricsenabled: true

# Enable wireguard encryption (future)
calico_wireguard_enabled: true

# Custom MTU
calico_veth_mtu: 1500

# Custom IP pool
calico_pool_cidr: 10.100.0.0/16
```

---

## Network Configuration

### IP Address Management (IPAM)

Calico uses block-based IPAM:

```
┌─────────────────────────────────────────────────────┐
│         Calico IPAM - Pod CIDR: 10.244.0.0/16       │
└─────────────────────────────────────────────────────┘

masternode (192.168.4.63)
├── Block: 10.244.0.0/26   (IPs: 10.244.0.1 - 10.244.0.62)
└── Pods: ~60 pods max

storagenodet3500 (192.168.4.61)
├── Block: 10.244.1.0/26   (IPs: 10.244.1.1 - 10.244.1.62)
└── Pods: ~60 pods max

homelab (192.168.4.62)
├── Block: 10.244.2.0/26   (IPs: 10.244.2.1 - 10.244.2.62)
└── Pods: ~60 pods max
```

### Routing Architecture

Calico uses **BGP (Border Gateway Protocol)** for route distribution:

```
┌──────────────────────────────────────────────────────────┐
│                 Calico BGP Routing                       │
└──────────────────────────────────────────────────────────┘

masternode (AS 64512)
  │
  │ BGP Peer
  ├────────────► storagenodet3500 (AS 64512)
  │
  │ BGP Peer
  └────────────► homelab (AS 64512)

Each node advertises:
- Local pod CIDR blocks
- Routes to reach pods on this node
```

### Traffic Flow

#### Pod-to-Pod Communication (Same Node)

```
Pod A (10.244.0.5) → veth pair → Linux bridge → veth pair → Pod B (10.244.0.10)
```

#### Pod-to-Pod Communication (Different Nodes)

```
Pod A (10.244.0.5) on masternode
  ↓ (via veth)
Host routing table
  ↓ (via BGP route: 10.244.1.0/26 via 192.168.4.61)
Physical network (192.168.4.0/24)
  ↓
storagenodet3500 (192.168.4.61)
  ↓ (via veth)
Pod B (10.244.1.10) on storagenodet3500
```

#### Pod-to-External Communication

```
Pod A (10.244.0.5)
  ↓ (SNAT to node IP)
Host network (192.168.4.63)
  ↓ (via default gateway)
Router (192.168.4.1)
  ↓
Internet
```

---

## Network Policies

### Default Policy Posture

By default, Calico allows all traffic. Network policies must be explicitly defined to restrict traffic.

### Example Network Policies

#### 1. Deny All Ingress (Default Deny)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

#### 2. Allow Monitoring Stack to Scrape Metrics

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-prometheus-scraping
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
      podSelector:
        matchLabels:
          app: prometheus
    ports:
    - protocol: TCP
      port: 8080
```

#### 3. Allow DNS Traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
```

### Calico Global Network Policies

Calico extends Kubernetes NetworkPolicy with **GlobalNetworkPolicy**:

```yaml
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: allow-cluster-internal
spec:
  order: 100
  ingress:
  - action: Allow
    source:
      nets:
      - 10.244.0.0/16  # Pod CIDR
      - 192.168.4.0/24  # Node network
  egress:
  - action: Allow
```

---

## Troubleshooting

### Common Issues

#### 1. Pods Unable to Communicate

**Symptoms**:
- Pod-to-pod communication fails
- DNS resolution fails
- Timeout errors

**Diagnosis**:
```bash
# Check Calico node status
kubectl get pods -n kube-system -l k8s-app=calico-node

# Check Calico node logs
kubectl logs -n kube-system -l k8s-app=calico-node

# Check BGP peer status
kubectl exec -n kube-system calico-node-<pod> -- calicoctl node status

# Check IP pool configuration
kubectl exec -n kube-system calico-node-<pod> -- calicoctl get ippool -o wide
```

**Solution**:
- Ensure all calico-node pods are running
- Verify IP pool configuration matches pod CIDR
- Check BGP peer connectivity

#### 2. IP Address Exhaustion

**Symptoms**:
- Pods stuck in `ContainerCreating`
- Error: "no IP addresses available in pool"

**Diagnosis**:
```bash
# Check IP pool usage
kubectl exec -n kube-system calico-node-<pod> -- calicoctl ipam show

# Check allocated blocks
kubectl exec -n kube-system calico-node-<pod> -- calicoctl ipam show --show-blocks
```

**Solution**:
- Increase IP pool size (requires cluster redeployment)
- Clean up unused IP addresses:
  ```bash
  kubectl exec -n kube-system calico-node-<pod> -- calicoctl ipam release --ip=<unused-ip>
  ```

#### 3. MTU Issues

**Symptoms**:
- Packet loss
- Connection timeouts
- Slow network performance

**Diagnosis**:
```bash
# Check MTU settings
ip link show | grep mtu

# Ping with large packet
ping -M do -s 1472 <target-ip>
```

**Solution**:
- Adjust Calico MTU in Kubespray inventory:
  ```yaml
  calico_veth_mtu: 1440  # For IP-in-IP
  calico_veth_mtu: 1450  # For VXLAN
  ```
- Redeploy cluster with updated MTU

#### 4. Network Policy Not Working

**Symptoms**:
- Traffic allowed when should be denied
- Traffic denied when should be allowed

**Diagnosis**:
```bash
# Check network policies
kubectl get networkpolicy -A

# Check Calico policy
kubectl exec -n kube-system calico-node-<pod> -- calicoctl get policy -o wide

# Check Felix logs for policy errors
kubectl logs -n kube-system -l k8s-app=calico-node -c calico-node | grep -i policy
```

**Solution**:
- Verify policy selectors match pod labels
- Check policy order (lower order = higher priority)
- Ensure policyTypes is set correctly

---

## Validation

### Validate Calico Installation

```bash
# 1. Check Calico components are running
kubectl get pods -n kube-system | grep calico

# Expected output:
# calico-kube-controllers-<hash>   1/1     Running
# calico-node-<hash>                1/1     Running (on each node)

# 2. Check Calico version
kubectl exec -n kube-system calico-node-<pod> -- calico-node --version

# 3. Check IP pools
kubectl exec -n kube-system calico-node-<pod> -- calicoctl get ippool -o wide

# Expected output:
# NAME                  CIDR            NAT    IPIPMODE   VXLANMODE
# default-ipv4-ippool   10.244.0.0/16   true   Always     Never

# 4. Check BGP status
kubectl exec -n kube-system calico-node-<pod> -- calicoctl node status

# Expected output:
# Calico process is running.
#
# IPv4 BGP status
# +--------------+-------------------+-------+----------+-------------+
# | PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
# +--------------+-------------------+-------+----------+-------------+
# | 192.168.4.61 | node-to-node mesh | up    | 12:00:00 | Established |
# | 192.168.4.62 | node-to-node mesh | up    | 12:00:00 | Established |
# +--------------+-------------------+-------+----------+-------------+
```

### Test Pod-to-Pod Connectivity

```bash
# 1. Create test pods
kubectl run test-pod-1 --image=busybox --command -- sleep 3600
kubectl run test-pod-2 --image=busybox --command -- sleep 3600

# 2. Get pod IPs
POD1_IP=$(kubectl get pod test-pod-1 -o jsonpath='{.status.podIP}')
POD2_IP=$(kubectl get pod test-pod-2 -o jsonpath='{.status.podIP}')

# 3. Test connectivity
kubectl exec test-pod-1 -- ping -c 3 $POD2_IP

# Expected output: 3 packets transmitted, 3 received

# 4. Cleanup
kubectl delete pod test-pod-1 test-pod-2
```

### Test DNS Resolution

```bash
# 1. Create test pod
kubectl run test-dns --image=busybox --command -- sleep 3600

# 2. Test DNS
kubectl exec test-dns -- nslookup kubernetes.default

# Expected output: DNS resolution successful

# 3. Cleanup
kubectl delete pod test-dns
```

### Test Network Policy

```bash
# 1. Create test namespace
kubectl create namespace netpol-test

# 2. Create test pods
kubectl run web -n netpol-test --image=nginx
kubectl run client -n netpol-test --image=busybox --command -- sleep 3600

# 3. Test connectivity (should work)
kubectl exec -n netpol-test client -- wget -O- --timeout=2 http://web

# 4. Apply deny-all policy
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: netpol-test
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF

# 5. Test connectivity (should fail)
kubectl exec -n netpol-test client -- wget -O- --timeout=2 http://web

# Expected: Connection timeout

# 6. Cleanup
kubectl delete namespace netpol-test
```

---

## Monitoring Calico

### Prometheus Metrics

Calico exports Prometheus metrics on port 9091:

```yaml
# Prometheus ServiceMonitor for Calico
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: calico-node
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: calico-node
  endpoints:
  - port: metrics
    interval: 30s
```

### Key Metrics to Monitor

- `felix_active_local_endpoints`: Number of endpoints managed by Felix
- `felix_active_local_policies`: Number of active policies
- `felix_iptables_restore_errors`: IPTables restore errors
- `felix_int_dataplane_failures`: Dataplane failures
- `felix_route_programming_duration_seconds`: Route programming latency

### Grafana Dashboard

Import Calico Grafana dashboard:
- Dashboard ID: 3244 (Felix Dashboard)
- URL: https://grafana.com/grafana/dashboards/3244

---

## Advanced Configuration

### Enable Prometheus Metrics

Edit Kubespray inventory:

```yaml
# cluster-infra/inventory/mycluster/group_vars/k8s_cluster/k8s-net-calico.yml
calico_felix_prometheusmetricsenabled: true
calico_felix_prometheusmetricsport: 9091
```

### Enable IP-in-IP Only for Cross-Subnet Traffic

```yaml
# Better performance when nodes are on same L2 network
calico_ipip_mode: 'CrossSubnet'
```

### Enable VXLAN Instead of IP-in-IP

```yaml
# Use when IP-in-IP is blocked by network
calico_vxlan_mode: 'Always'
calico_ipip_mode: 'Never'
```

### Enable WireGuard Encryption (Future)

```yaml
# Encrypt pod-to-pod traffic
calico_wireguard_enabled: true
```

---

## References

- **Calico Documentation**: https://docs.tigera.io/calico/latest/about/
- **Kubespray Calico Configuration**: https://github.com/kubernetes-sigs/kubespray/blob/master/docs/calico.md
- **Calico Network Policy**: https://docs.tigera.io/calico/latest/network-policy/
- **Calico Architecture**: https://docs.tigera.io/calico/latest/reference/architecture/
- **Troubleshooting Guide**: https://docs.tigera.io/calico/latest/operations/troubleshoot/

---

## Quick Reference Commands

```bash
# Get Calico version
kubectl exec -n kube-system calico-node-<pod> -- calico-node --version

# Check node status
kubectl exec -n kube-system calico-node-<pod> -- calicoctl node status

# Get IP pools
kubectl exec -n kube-system calico-node-<pod> -- calicoctl get ippool -o wide

# Check BGP peers
kubectl exec -n kube-system calico-node-<pod> -- calicoctl get bgppeer

# Check IP allocation
kubectl exec -n kube-system calico-node-<pod> -- calicoctl ipam show

# Get network policies
kubectl get networkpolicy -A

# Get Calico policies
kubectl exec -n kube-system calico-node-<pod> -- calicoctl get policy -o wide

# Check Felix configuration
kubectl exec -n kube-system calico-node-<pod> -- calicoctl get felixconfig -o yaml
```

---

**Last Updated**: November 30, 2025  
**Maintained By**: VMStation Infrastructure Team  
**Related Documentation**: cluster-docs/ARCHITECTURE.md, cluster-docs/CAPABILITY_PARITY_ANALYSIS.md
