# Configuration Reference

Configuration options for VMStation components.

## Inventory Configuration

### hosts.yml

```yaml
all:
  vars:
    kubernetes_version: "1.29"
    pod_network_cidr: "10.244.0.0/16"
    service_network_cidr: "10.96.0.0/12"
    cni_plugin: flannel

monitoring_nodes:
  hosts:
    masternode:
      ansible_host: 192.168.4.63
      ansible_connection: local

storage_nodes:
  hosts:
    storagenodet3500:
      ansible_host: 192.168.4.61
      ansible_user: root
      ansible_ssh_private_key_file: /root/.ssh/id_k3s
      mac_address: "b8:ac:6f:7e:6c:9d"

compute_nodes:
  hosts:
    homelab:
      ansible_host: 192.168.4.62
      ansible_user: jashandeepjustinbains
      ansible_ssh_private_key_file: /root/.ssh/id_k3s
      mac_address: "d0:94:66:30:d6:63"
```

### Key Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_version` | "1.29" | Kubernetes version |
| `pod_network_cidr` | "10.244.0.0/16" | Pod network CIDR |
| `service_network_cidr` | "10.96.0.0/12" | Service network CIDR |
| `cni_plugin` | "flannel" | CNI plugin (flannel/calico) |

## Prometheus Configuration

### prometheus.yaml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets:
        - '192.168.4.63:9100'
        - '192.168.4.61:9100'
        - '192.168.4.62:9100'
```

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `scrape_interval` | 15s | Metric collection interval |
| `retention.time` | 15d | Data retention period |
| `retention.size` | 8GB | Maximum storage size |

## Loki Configuration

### loki.yaml

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
  filesystem:
    directory: /loki/chunks

limits_config:
  retention_period: 744h  # 31 days
```

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `http_listen_port` | 3100 | HTTP port |
| `replication_factor` | 1 | Replication (single instance) |
| `retention_period` | 744h | Log retention |
| `period` | 24h | Index period (required) |

## Grafana Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `GF_AUTH_ANONYMOUS_ENABLED` | true | Enable anonymous access |
| `GF_AUTH_ANONYMOUS_ORG_ROLE` | Viewer | Anonymous user role |
| `GF_SECURITY_ADMIN_PASSWORD` | admin | Admin password |
| `GF_INSTALL_PLUGINS` | - | Plugins to install |

### Datasource Configuration

```yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    access: proxy
    isDefault: true
  - name: Loki
    type: loki
    url: http://loki:3100
    access: proxy
```

## Storage Configuration

### StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
```

### PVC Sizes

| Component | Size | Purpose |
|-----------|------|---------|
| Prometheus | 10Gi | Metrics storage |
| Loki | 20Gi | Log storage |
| Grafana | 2Gi | Dashboards, config |

## Chrony Configuration

### chrony.conf

```
server pool.ntp.org iburst
server time.cloudflare.com iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
```

## Preflight Role Variables

### defaults/main.yml

```yaml
preflight_chrony_servers:
  - pool.ntp.org
  - time.cloudflare.com

preflight_firewall_ports:
  - 6443/tcp
  - 10250/tcp
  - 30000-32767/tcp

preflight_selinux_mode: permissive

preflight_kernel_modules:
  - br_netfilter
  - overlay
  - ip_vs

preflight_sysctl_settings:
  net.bridge.bridge-nf-call-iptables: 1
  net.ipv4.ip_forward: 1
```

## Related Documentation

- [Installation](../getting-started/installation.md)
- [Deployment](../deployment/README.md)
- [Architecture](../architecture/README.md)
