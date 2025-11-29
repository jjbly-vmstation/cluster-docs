# Glossary

Terms and definitions used in VMStation documentation.

## A

### Ansible
Configuration management and automation tool used for deploying VMStation.

### Auto-sleep
Feature that automatically suspends idle worker nodes to save power.

## C

### Calico
Container Network Interface (CNI) plugin providing networking and network policy for Kubernetes.

### Chrony
NTP client/server for time synchronization.

### CNI (Container Network Interface)
Plugin framework for configuring network interfaces in Linux containers.

### Control Plane
The components that manage the Kubernetes cluster (API server, scheduler, controller manager, etcd).

### CoreDNS
DNS server for service discovery within Kubernetes.

## D

### DaemonSet
Kubernetes workload that runs a pod on every node (e.g., node-exporter, promtail).

### Deployment
Kubernetes workload for stateless applications with declarative updates.

## E

### etcd
Distributed key-value store used by Kubernetes to store cluster state.

### Exporter
Application that exposes metrics in Prometheus format.

## F

### Flannel
Simple overlay network for Kubernetes.

### FreeIPA
Identity management system providing LDAP, Kerberos, and DNS.

## G

### Grafana
Visualization and dashboarding platform for metrics and logs.

## H

### HostPath
Kubernetes volume type that mounts a file or directory from the host node.

## K

### kubeadm
Tool for bootstrapping Kubernetes clusters.

### kubectl
Command-line tool for interacting with Kubernetes.

### Kubespray
Ansible-based Kubernetes cluster deployment tool.

### Kubelet
Node agent that manages pods and containers.

## L

### Local Path Provisioner
Dynamic storage provisioner for local storage.

### Loki
Log aggregation system from Grafana Labs.

### LogQL
Query language for Loki logs.

## M

### Manifest
YAML file defining Kubernetes resources.

### masternode
Control plane node in VMStation (192.168.4.63).

## N

### Namespace
Virtual cluster within Kubernetes for resource isolation.

### Node Exporter
Prometheus exporter for system-level metrics.

### NodePort
Service type that exposes a port on all cluster nodes.

## P

### PersistentVolume (PV)
Cluster-level storage resource.

### PersistentVolumeClaim (PVC)
Request for storage by a pod.

### Pod
Smallest deployable unit in Kubernetes, containing one or more containers.

### Prometheus
Metrics collection and alerting system.

### PromQL
Query language for Prometheus metrics.

### Promtail
Log shipper that sends logs to Loki.

## R

### RBAC
Role-Based Access Control for Kubernetes authorization.

### RKE2
Rancher's next-generation Kubernetes distribution.

## S

### SecurityContext
Pod/container-level security settings.

### Service
Kubernetes resource providing stable networking for pods.

### StatefulSet
Kubernetes workload for stateful applications with stable identities.

### StorageClass
Defines a class of storage for dynamic provisioning.

### storagenodet3500
Worker node in VMStation (192.168.4.61).

## T

### Taint
Node attribute that repels pods without matching tolerations.

### Toleration
Pod attribute that allows scheduling on tainted nodes.

## V

### Volume
Directory accessible to containers in a pod.

## W

### Wake-on-LAN (WoL)
Technology to remotely power on computers via network.

### Worker Node
Cluster node that runs application workloads.

## Related Documentation

- [Architecture Overview](../architecture/overview.md)
- [Getting Started](../getting-started/README.md)
