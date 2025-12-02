# VMStation-Org Deployment Guide

**Date**: November 30, 2025  
**Version**: 1.0.0  
**Purpose**: Step-by-step deployment guide for vmstation-org modular Kubernetes cluster

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Deployment Overview](#deployment-overview)
3. [Phase 1: Bootstrap](#phase-1-bootstrap)
4. [Phase 2: Infrastructure Provisioning](#phase-2-infrastructure-provisioning)
5. [Phase 3: System Configuration](#phase-3-system-configuration)
6. [Phase 4: Monitoring Stack](#phase-4-monitoring-stack)
7. [Phase 5: Application Deployment](#phase-5-application-deployment)
8. [Phase 6: Validation](#phase-6-validation)
9. [Troubleshooting](#troubleshooting)
10. [Rollback Procedures](#rollback-procedures)

---

## Prerequisites

### Development Environment

**Location**: Windows 11 development machine  
**Target**: Linux Kubernetes cluster (Debian 12, RHEL 10)

**Required Tools**:
- Git
- SSH client (OpenSSH)
- Ansible 2.15+ (WSL or native Windows)
- Python 3.10+
- kubectl 1.29+
- Text editor (VSCode recommended)

### Target Infrastructure

**Control Plane (Always-On)**:
- **masternode**: 192.168.4.63 (Debian 12)
  - Role: Kubernetes control plane, bastion, DRONE CI/CD
  - SSH: Configured with public key authentication
  - Always powered on

**Worker Nodes**:
- **storagenodet3500**: 192.168.4.61 (Debian 12)
  - Role: Storage worker, auto-sleep enabled
  - Wake-on-LAN: Supported

- **homelab**: 192.168.4.62 (RHEL 10)
  - Role: Compute worker, on-demand
  - Deployment: Kubespray

### Network Configuration

- **Network**: 192.168.4.0/24
- **Gateway**: 192.168.4.1
- **DNS**: Configure on router or use public DNS
- **Ports**: Ensure Kubernetes ports open (6443, 10250, etc.)

### Repository Setup

All repositories cloned to: `f:\vmstation-org\`

```
f:\vmstation-org\
├── cluster-docs/
├── cluster-infra/
├── cluster-config/
├── cluster-setup/
├── cluster-monitor-stack/
├── cluster-application-stack/
└── cluster-tools/
```

### SSH Key Configuration

**Bastion**: masternode (192.168.4.63)

**From Windows**:
```powershell
# Copy SSH public key to masternode
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh user@192.168.4.63 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Test SSH access
ssh user@192.168.4.63
```

**From masternode to all nodes**:
```bash
# On masternode, generate SSH key if not exists
ssh-keygen -t rsa -b 4096

# Copy to all nodes
ssh-copy-id user@192.168.4.61  # storagenodet3500
ssh-copy-id user@192.168.4.62  # homelab

# Test SSH access
ssh user@192.168.4.61
ssh user@192.168.4.62
```

---

## Deployment Overview

### Deployment Phases

```
Phase 1: Bootstrap (cluster-setup)
   ↓
Phase 2: Infrastructure Provisioning (cluster-infra)
   ↓
Phase 3: System Configuration (cluster-config)
   ↓
Phase 4: Monitoring Stack (cluster-monitor-stack)
   ↓
Phase 5: Application Deployment (cluster-application-stack)
   ↓
Phase 6: Validation (cluster-tools)
```

### Estimated Time

| Phase | Estimated Time |
|-------|---------------|
| Phase 1: Bootstrap | 15-30 minutes |
| Phase 2: Infrastructure | 30-60 minutes |
| Phase 3: Configuration | 20-40 minutes |
| Phase 4: Monitoring | 15-30 minutes |
| Phase 5: Applications | 10-20 minutes |
| Phase 6: Validation | 10-15 minutes |
| **Total** | **2-3 hours** |

### Deployment Checklist

- [ ] Prerequisites verified
- [ ] SSH keys configured
- [ ] Repositories cloned
- [ ] Network connectivity tested
- [ ] Phase 1: Bootstrap complete
- [ ] Phase 2: Infrastructure complete
- [ ] Phase 3: Configuration complete
- [ ] Phase 4: Monitoring complete
- [ ] Phase 5: Applications complete
- [ ] Phase 6: Validation complete

---

## Phase 1: Bootstrap

**Purpose**: Prepare nodes for cluster deployment  
**Repository**: `cluster-setup`  
**Estimated Time**: 15-30 minutes

### Step 1.1: Install Dependencies

From Windows (PowerShell):

```powershell
# SSH to masternode
ssh user@192.168.4.63

# On masternode
cd ~/vmstation-org/cluster-setup/bootstrap

# Install Ansible and dependencies
./install-dependencies.sh
```

**Expected Output**:
```
✓ Python 3.10+ installed
✓ Ansible 2.15+ installed
✓ Required Python packages installed
✓ kubectl installed
Bootstrap dependencies installation complete
```

### Step 1.2: Setup SSH Keys

```bash
# On masternode
cd ~/vmstation-org/cluster-setup/bootstrap

# Setup SSH keys for all nodes
./setup-ssh-keys.sh

# Verify SSH access
ssh storagenodet3500.local 'hostname'
ssh homelab.local 'hostname'
```

**Expected Output**:
```
✓ SSH keys distributed to all nodes
✓ Passwordless authentication configured
SSH setup complete
```

### Step 1.3: Prepare Nodes

```bash
# On masternode
cd ~/vmstation-org/cluster-setup/bootstrap

# Prepare all nodes (OS updates, packages, etc.)
./prepare-nodes.sh
```

**Expected Output**:
```
✓ masternode prepared
✓ storagenodet3500 prepared
✓ homelab prepared
Node preparation complete
```

### Step 1.4: Verify Prerequisites

```bash
# On masternode
cd ~/vmstation-org/cluster-setup/bootstrap

# Verify all prerequisites
./verify-prerequisites.sh
```

**Expected Output**:
```
✓ All nodes reachable
✓ SSH authentication working
✓ Required packages installed
✓ Network connectivity verified
✓ Disk space sufficient
Prerequisites verification complete
```

### Checkpoint 1

- [ ] Dependencies installed on masternode
- [ ] SSH keys configured for all nodes
- [ ] All nodes prepared
- [ ] Prerequisites verified

---

## Phase 2: Infrastructure Provisioning

**Purpose**: Deploy Kubernetes cluster using Kubespray  
**Repository**: `cluster-infra`  
**Estimated Time**: 30-60 minutes

### Step 2.1: Configure Kubespray Inventory

```bash
# On masternode
cd ~/vmstation-org/cluster-infra/inventory/mycluster

# Review and edit inventory
vim inventory.ini
```

**inventory.ini Example**:
```ini
[all]
masternode ansible_host=192.168.4.63 ip=192.168.4.63
storagenodet3500 ansible_host=192.168.4.61 ip=192.168.4.61
homelab ansible_host=192.168.4.62 ip=192.168.4.62

[kube_control_plane]
masternode

[etcd]
masternode

[kube_node]
storagenodet3500
homelab

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

### Step 2.2: Configure Calico CNI

```bash
# Edit Calico configuration
vim group_vars/k8s_cluster/k8s-net-calico.yml
```

**k8s-net-calico.yml Recommended Settings**:
```yaml
# CNI Plugin
kube_network_plugin: calico

# Calico Configuration
calico_ipip_mode: 'Never'  # Same L2 network, no encapsulation needed
calico_vxlan_mode: 'Never'
calico_network_backend: vxlan

# IP Pools
calico_pool_cidr: 10.244.0.0/16
calico_pool_blocksize: 26

# MTU
calico_veth_mtu: 1500

# Prometheus metrics
calico_felix_prometheusmetricsenabled: true
calico_felix_prometheusmetricsport: 9091
```

### Step 2.3: Run Preflight Checks (RHEL10)

```bash
# On masternode
cd ~/vmstation-org/cluster-infra/ansible

# Run RHEL10 preflight role
ansible-playbook playbooks/run-preflight-rhel10.yml
```

**Expected Output**:
```
✓ RHEL10 repositories configured
✓ Required packages installed
✓ Firewall rules configured
✓ SELinux configured
RHEL10 preflight complete
```

### Step 2.4: Deploy Kubernetes Cluster

```bash
# On masternode
cd ~/vmstation-org/cluster-infra/scripts

# Deploy cluster using Kubespray wrapper
./run-kubespray.sh deploy
```

**This will**:
1. Install container runtime (containerd)
2. Deploy Kubernetes control plane
3. Join worker nodes
4. Deploy Calico CNI
5. Configure kube-proxy
6. Deploy CoreDNS

**Expected Duration**: 30-60 minutes

**Expected Output**:
```
PLAY RECAP *********************************************************************
masternode                 : ok=XXX  changed=XXX  unreachable=0    failed=0
storagenodet3500           : ok=XXX  changed=XXX  unreachable=0    failed=0
homelab                    : ok=XXX  changed=XXX  unreachable=0    failed=0

Kubernetes cluster deployed successfully
```

### Step 2.5: Verify Cluster

```bash
# On masternode
kubectl get nodes

# Expected output:
# NAME                STATUS   ROLES           AGE   VERSION
# masternode          Ready    control-plane   5m    v1.29.x
# storagenodet3500    Ready    <none>          4m    v1.29.x
# homelab             Ready    <none>          4m    v1.29.x

kubectl get pods -A

# Expected: All pods in Running state
```

### Checkpoint 2

- [ ] Kubespray inventory configured
- [ ] Calico CNI configured
- [ ] RHEL10 preflight complete
- [ ] Kubernetes cluster deployed
- [ ] All nodes in Ready state
- [ ] All system pods running

---

## Phase 3: System Configuration

**Purpose**: Apply baseline configuration and deploy infrastructure services  
**Repository**: `cluster-config`  
**Estimated Time**: 20-40 minutes

### Step 3.1: Review Inventory

```bash
# On masternode
cd ~/vmstation-org/cluster-config

# Review inventory
cat /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml

# Expected:
# [all]
# masternode ansible_host=192.168.4.63
# storagenodet3500 ansible_host=192.168.4.61
# homelab ansible_host=192.168.4.62
```

### Step 3.2: Apply Baseline Configuration

```bash
# On masternode
cd ~/vmstation-org/cluster-config/ansible

# Apply full configuration
ansible-playbook playbooks/site.yml

# Or apply specific roles:
# ansible-playbook playbooks/site.yml --tags ssh
# ansible-playbook playbooks/site.yml --tags networking
# ansible-playbook playbooks/site.yml --tags storage
```

**This will**:
- Apply SSH hardening
- Configure network settings
- Mount storage volumes
- Apply sysctl settings
- Configure systemd services

**Expected Output**:
```
PLAY RECAP *********************************************************************
masternode                 : ok=XX  changed=XX  unreachable=0    failed=0
storagenodet3500           : ok=XX  changed=XX  unreachable=0    failed=0
homelab                    : ok=XX  changed=XX  unreachable=0    failed=0

Baseline configuration applied successfully
```

### Step 3.3: Deploy Infrastructure Services

```bash
# On masternode
cd ~/vmstation-org/cluster-config/ansible

# Deploy all infrastructure services
ansible-playbook playbooks/infrastructure-services.yml

# Or deploy individually:
# ansible-playbook playbooks/ntp-sync.yml
# ansible-playbook playbooks/syslog-server.yml
# ansible-playbook playbooks/kerberos-setup.yml
```

**This will deploy**:
- NTP/Chrony time synchronization
- Centralized syslog server
- Kerberos/FreeIPA (optional)

**Expected Output**:
```
PLAY RECAP *********************************************************************
Kubernetes playbook completed successfully

Infrastructure services deployed:
✓ NTP/Chrony
✓ Syslog server
✓ Kerberos/FreeIPA
```

### Step 3.4: Verify Configuration

```bash
# On masternode
cd ~/vmstation-org/cluster-config/scripts

# Verify baseline configuration
./check-drift.sh

# Expected: No configuration drift detected
```

### Checkpoint 3

- [ ] Baseline configuration applied
- [ ] Infrastructure services deployed
- [ ] NTP synchronization working
- [ ] Syslog centralization working
- [ ] No configuration drift detected

---

## Phase 4: Monitoring Stack

**Purpose**: Deploy observability stack (Prometheus, Grafana, Loki)  
**Repository**: `cluster-monitor-stack`  
**Estimated Time**: 15-30 minutes

### Step 4.1: Deploy Monitoring Stack

```bash
# On masternode
cd ~/vmstation-org/cluster-monitor-stack/ansible

# Deploy full monitoring stack
ansible-playbook playbooks/deploy-monitoring-stack.yaml
```

**This will deploy**:
- Prometheus (metrics collection)
- Grafana (visualization)
- Loki (log aggregation)
- Promtail (log shipping)
- Node Exporter (system metrics)
- Kube-state-metrics (K8s metrics)
- IPMI Exporter (hardware metrics)

**Expected Output**:
```
PLAY RECAP *********************************************************************
Monitoring stack deployed successfully

Services exposed:
- Grafana: http://192.168.4.63:30001
- Prometheus: http://192.168.4.63:30000
- Loki: http://192.168.4.63:3100
```

### Step 4.2: Verify Monitoring Stack

```bash
# Check all monitoring pods
kubectl get pods -n monitoring

# Expected: All pods in Running state
# prometheus-0                 1/1     Running
# grafana-xxxx                 1/1     Running
# loki-0                       1/1     Running
# promtail-xxxx (DaemonSet)    1/1     Running per node
# node-exporter-xxxx           1/1     Running per node
```

### Step 4.3: Access Grafana

**From Windows**:

1. Open browser: http://192.168.4.63:30001
2. Login with default credentials (admin/admin)
3. Change password when prompted
4. Verify dashboards visible

**Import Calico Dashboard**:
- Dashboard ID: 3244
- Name: "Felix Dashboard by Calico"

### Step 4.4: Verify Metrics Collection

```bash
# On masternode
# Check Prometheus targets
curl http://192.168.4.63:30000/api/v1/targets

# Check Loki logs
curl http://192.168.4.63:3100/loki/api/v1/labels
```

### Checkpoint 4

- [ ] Monitoring stack deployed
- [ ] All monitoring pods running
- [ ] Grafana accessible
- [ ] Prometheus collecting metrics
- [ ] Loki collecting logs
- [ ] Dashboards imported

---

## Phase 5: Application Deployment

**Purpose**: Deploy application workloads (Jellyfin, etc.)  
**Repository**: `cluster-application-stack`  
**Estimated Time**: 10-20 minutes

### Step 5.1: Deploy Jellyfin

```bash
# On masternode
cd ~/vmstation-org/cluster-application-stack/ansible

# Deploy Jellyfin media server
ansible-playbook playbooks/jellyfin.yml
```

**Expected Output**:
```
PLAY RECAP *********************************************************************
Jellyfin deployed successfully

Service exposed:
- Jellyfin: http://192.168.4.63:30002
```

### Step 5.2: Verify Jellyfin

```bash
# Check Jellyfin pod
kubectl get pods -l app=jellyfin

# Expected: jellyfin-xxxx   1/1     Running

# Check service
kubectl get svc jellyfin

# Expected: TYPE=NodePort, PORT=30002
```

### Step 5.3: Access Jellyfin

**From Windows**:

1. Open browser: http://192.168.4.63:30002
2. Complete Jellyfin initial setup wizard
3. Configure media libraries

### Step 5.4: Deploy Additional Applications (Optional)

```bash
# On masternode
cd ~/vmstation-org/cluster-application-stack/ansible

# Deploy Minecraft server (optional)
ansible-playbook playbooks/deploy-minecraft.yml
```

### Checkpoint 5

- [ ] Jellyfin deployed
- [ ] Jellyfin accessible
- [ ] Applications configured
- [ ] All application pods running

---

## Phase 6: Validation

**Purpose**: Validate entire cluster deployment  
**Repository**: `cluster-tools`  
**Estimated Time**: 10-15 minutes

### Step 6.1: Validate Cluster Health

```bash
# On masternode
cd ~/vmstation-org/cluster-tools/validation

# Run comprehensive health check
./validate-cluster-health.sh
```

**Expected Output**:
```
✓ All nodes ready
✓ All system pods running
✓ All application pods running
✓ etcd healthy
✓ Control plane healthy

Cluster health: PASS
```

### Step 6.2: Validate Monitoring Stack

```bash
# On masternode
cd ~/vmstation-org/cluster-tools/validation

# Validate monitoring
./validate-monitoring-stack.sh
```

**Expected Output**:
```
✓ Prometheus accessible
✓ Grafana accessible
✓ Loki accessible
✓ All targets scraped
✓ Metrics collected
✓ Logs aggregated

Monitoring stack: PASS
```

### Step 6.3: Validate Network Connectivity

```bash
# On masternode
cd ~/vmstation-org/cluster-tools/validation

# Validate networking
./validate-network-connectivity.sh
```

**Expected Output**:
```
✓ Pod-to-pod communication working
✓ Pod-to-service communication working
✓ DNS resolution working
✓ External connectivity working
✓ Calico BGP peers established

Network connectivity: PASS
```

### Step 6.4: Run Integration Tests

```bash
# On masternode
cd ~/vmstation-org/cluster-tools/tests/integration

# Test sleep-wake cycle (if power management configured)
./test-sleep-wake-cycle.sh
```

### Step 6.5: Pre-Deployment Checklist

```bash
# On masternode
cd ~/vmstation-org/cluster-tools/validation

# Final validation checklist
./pre-deployment-checklist.sh
```

**Expected Output**:
```
Pre-Deployment Checklist:

✓ Kubernetes cluster operational
✓ All nodes ready
✓ CNI (Calico) working
✓ DNS resolution working
✓ Monitoring stack deployed
✓ Logging stack deployed
✓ Application workloads deployed
✓ Network connectivity validated
✓ Configuration baseline applied
✓ No configuration drift detected

DEPLOYMENT VALIDATION: PASS

Cluster is production-ready!
```

### Checkpoint 6

- [ ] Cluster health validated
- [ ] Monitoring stack validated
- [ ] Network connectivity validated
- [ ] Integration tests passed
- [ ] Pre-deployment checklist passed
- [ ] **DEPLOYMENT COMPLETE** ✅

---

## Troubleshooting

### Common Issues

#### 1. Node Not Ready

**Symptoms**:
```
NAME                STATUS     ROLES           AGE   VERSION
homelab             NotReady   <none>          5m    v1.29.x
```

**Diagnosis**:
```bash
kubectl describe node homelab
kubectl get pods -n kube-system -o wide | grep homelab
```

**Common Causes**:
- CNI not running on node
- kubelet not started
- Network connectivity issues

**Solution**:
```bash
# Check kubelet status
ssh homelab 'systemctl status kubelet'

# Check CNI pods
kubectl get pods -n kube-system -l k8s-app=calico-node

# Restart kubelet if needed
ssh homelab 'systemctl restart kubelet'
```

#### 2. Pods Not Starting

**Symptoms**:
```
NAME                     READY   STATUS              RESTARTS   AGE
my-pod                   0/1     ContainerCreating   0          5m
```

**Diagnosis**:
```bash
kubectl describe pod my-pod
kubectl get events --sort-by='.lastTimestamp'
```

**Common Causes**:
- No IP addresses available (IPAM exhausted)
- Image pull failure
- PersistentVolume not available

**Solution**:
```bash
# Check IP allocation
kubectl exec -n kube-system calico-node-xxxx -- calicoctl ipam show

# Check image pull
kubectl describe pod my-pod | grep -A 5 Events

# Check PV/PVC
kubectl get pv,pvc
```

#### 3. Monitoring Not Accessible

**Symptoms**:
- Cannot access Grafana at http://192.168.4.63:30001
- Connection timeout

**Diagnosis**:
```bash
# Check monitoring pods
kubectl get pods -n monitoring

# Check services
kubectl get svc -n monitoring

# Check NodePort
kubectl get svc grafana -n monitoring -o yaml | grep nodePort
```

**Solution**:
```bash
# Redeploy monitoring if needed
cd ~/vmstation-org/cluster-monitor-stack/ansible
ansible-playbook playbooks/deploy-monitoring-stack.yaml

# Check firewall rules
ssh masternode 'sudo iptables -L -n | grep 30001'
```

#### 4. Ansible Playbook Failures

**Symptoms**:
```
fatal: [homelab]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect"}
```

**Diagnosis**:
```bash
# Test SSH connectivity
ssh homelab 'hostname'

# Test Ansible connectivity
ansible all -m ping -i /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml
```

**Solution**:
```bash
# Re-run SSH key setup
cd ~/vmstation-org/cluster-setup/bootstrap
./setup-ssh-keys.sh

# Verify SSH config
cat ~/.ssh/config
```

---

## Rollback Procedures

### Rollback Kubespray Deployment

```bash
# On masternode
cd ~/vmstation-org/cluster-infra/scripts

# Reset cluster
./run-kubespray.sh reset

# WARNING: This will delete the entire cluster
# Confirm when prompted
```

### Rollback Configuration Changes

```bash
# On masternode
cd ~/vmstation-org/cluster-config

# Restore from baseline
git checkout HEAD~1 hosts/masternode/sshd_config

# Re-apply configuration
ansible-playbook playbooks/site.yml
```

### Rollback Monitoring Stack

```bash
# On masternode
# Delete monitoring namespace
kubectl delete namespace monitoring

# Redeploy previous version
cd ~/vmstation-org/cluster-monitor-stack
git checkout <previous-commit>
ansible-playbook playbooks/deploy-monitoring-stack.yaml
```

---

## Post-Deployment Tasks

### Configure Power Management

```bash
# On masternode
cd ~/vmstation-org/cluster-setup/power-management/playbooks

# Setup auto-sleep for storagenodet3500
ansible-playbook setup-autosleep.yaml

# Deploy Wake-on-LAN event handler
ansible-playbook deploy-event-wake.yaml
```

### Setup Automated Backups

```bash
# On masternode
# Setup etcd backup cron job
kubectl apply -f ~/vmstation-org/cluster-config/manifests/backup/etcd-backup-cronjob.yaml

# Verify backup job
kubectl get cronjob -n kube-system
```

### Configure Alerting

```bash
# On masternode
cd ~/vmstation-org/cluster-monitor-stack

# Apply alerting rules
kubectl apply -f manifests/prometheus/alerting-rules.yaml

# Configure Alertmanager (optional)
kubectl apply -f manifests/prometheus/alertmanager.yaml
```

---

## Daily Operations

### Check Cluster Status

```bash
# On masternode
kubectl get nodes
kubectl get pods -A
kubectl top nodes
kubectl top pods -A
```

### Access Dashboards

- **Grafana**: http://192.168.4.63:30001
- **Prometheus**: http://192.168.4.63:30000
- **Kubernetes Dashboard** (if deployed): http://192.168.4.63:30080

### Wake Sleeping Node

```bash
# On masternode
cd ~/vmstation-org/cluster-tools/power-management

# Wake storagenodet3500
./vmstation-event-wake.sh storagenodet3500

# Check power state
./check-power-state.sh
```

### Check for Drift

```bash
# On masternode
cd ~/vmstation-org/cluster-tools/tests/drift-detection

# Run drift detection
./check-drift.sh
```

---

## Support & Documentation

### Documentation References

- **Architecture**: `cluster-docs/ARCHITECTURE.md`
- **CNI Configuration**: `cluster-docs/CNI_CALICO_CONFIGURATION.md`
- **Capability Parity**: `cluster-docs/CAPABILITY_PARITY_ANALYSIS.md`
- **Final Validation**: `cluster-docs/FINAL_VALIDATION_REPORT.md`

### Getting Help

- **GitHub Issues**: Submit to appropriate component repository
- **Documentation**: Search `cluster-docs/`
- **Logs**: Check `kubectl logs` and Grafana/Loki

---

**Deployment Guide Version**: 1.0.0  
**Last Updated**: November 30, 2025  
**Maintained By**: VMStation Infrastructure Team
