# Glossary

Terminology used in VMStation documentation.

## A

**Ansible**
: Configuration management tool used for deployment automation.

**Auto-sleep**
: Feature that automatically suspends idle worker nodes to save power.

## C

**Calico**
: Container Network Interface (CNI) plugin providing networking and network policy.

**CNI (Container Network Interface)**
: Standard for configuring network interfaces in Linux containers.

**ConfigMap**
: Kubernetes resource for storing non-sensitive configuration data.

**CoreDNS**
: DNS server providing service discovery within the cluster.

**CrashLoopBackOff**
: Pod status indicating repeated container crashes.

## D

**DaemonSet**
: Kubernetes workload that runs a pod on every node.

**Deployment**
: Kubernetes workload for managing stateless applications.

## E

**etcd**
: Key-value store used by Kubernetes for cluster state.

**Exporter**
: Service that exposes metrics in Prometheus format.

## F

**Flannel**
: Simple CNI plugin using overlay networking.

**FreeIPA**
: Identity management system providing LDAP, Kerberos, and DNS.

## G

**Grafana**
: Open-source visualization and dashboarding platform.

## H

**HostPath**
: Volume type that mounts a host directory into pods.

## I

**IPMI**
: Intelligent Platform Management Interface for hardware monitoring.

## K

**Kerberos**
: Network authentication protocol.

**Kubelet**
: Primary node agent that runs pods on each node.

**Kubernetes**
: Container orchestration platform.

**Kubespray**
: Ansible-based tool for deploying Kubernetes clusters.

**kubeadm**
: Official Kubernetes cluster bootstrapping tool.

## L

**Loki**
: Log aggregation system optimized for Kubernetes.

**LogQL**
: Query language for Loki logs.

## M

**Manifest**
: YAML file defining Kubernetes resources.

## N

**Namespace**
: Virtual cluster for resource isolation.

**Node**
: Physical or virtual machine in the cluster.

**Node Exporter**
: Prometheus exporter for system metrics.

**NodePort**
: Service type exposing pods on a static port on each node.

## P

**Pod**
: Smallest deployable unit in Kubernetes.

**PromQL**
: Query language for Prometheus metrics.

**Prometheus**
: Open-source monitoring and alerting system.

**Promtail**
: Log shipping agent for Loki.

**PV (PersistentVolume)**
: Cluster-wide storage resource.

**PVC (PersistentVolumeClaim)**
: Request for storage by a pod.

## R

**RBAC (Role-Based Access Control)**
: Authorization mechanism for Kubernetes.

**RKE2**
: Rancher Kubernetes Engine 2, a Kubernetes distribution.

## S

**Secret**
: Kubernetes resource for storing sensitive data.

**Service**
: Kubernetes resource for exposing pods.

**StatefulSet**
: Kubernetes workload for managing stateful applications.

**StorageClass**
: Defines storage provisioning parameters.

## T

**Taint**
: Node attribute that repels pods without matching tolerations.

**Toleration**
: Pod attribute allowing scheduling on tainted nodes.

**TSDB**
: Time-Series Database (Prometheus storage).

## V

**Volume**
: Storage attached to pods.

## W

**WoL (Wake-on-LAN)**
: Technology to wake computers remotely via network.

## Related Documentation

- [Getting Started](../getting-started/README.md)
- [Architecture](../architecture/README.md)
