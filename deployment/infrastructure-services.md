# Infrastructure Services Deployment

This guide covers deploying core infrastructure services on VMStation.

## Infrastructure Components

| Service | Purpose | Deployment Type |
|---------|---------|-----------------|
| NTP/Chrony | Time synchronization | DaemonSet |
| Syslog | Centralized logging | Deployment |
| Kerberos/FreeIPA | Identity management | StatefulSet (optional) |

## Quick Deployment

```bash
./deploy.sh infrastructure
```

## NTP/Chrony

### Purpose

Ensures all cluster nodes have synchronized time - critical for:
- Kubernetes certificates
- Log correlation
- Monitoring accuracy

### Deployment

```bash
kubectl apply -f manifests/infrastructure/chrony.yaml
```

### Configuration

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: chrony-ntp
  namespace: infrastructure
spec:
  selector:
    matchLabels:
      app: chrony-ntp
  template:
    metadata:
      labels:
        app: chrony-ntp
    spec:
      hostNetwork: true
      containers:
      - name: chrony
        image: cturra/ntp:latest
        securityContext:
          capabilities:
            add: ["SYS_TIME"]
        volumeMounts:
        - name: chrony-conf
          mountPath: /etc/chrony/chrony.conf
          subPath: chrony.conf
      volumes:
      - name: chrony-conf
        configMap:
          name: chrony-config
```

### Verify Time Sync

```bash
# Check chrony pods
kubectl get pods -n infrastructure -l app=chrony-ntp

# Check time on node
chronyc tracking

# Run validation
./tests/validate-time-sync.sh
```

## Syslog Server

### Purpose

Centralized log collection from:
- Node system logs
- Application logs
- Network device logs

### Current Status

Syslog collection can be handled by:
1. **Promtail** (already deployed)
2. **Dedicated syslog server** (optional)

### Using Promtail for Syslog

Promtail already collects system logs. Configure syslog scraping:

```yaml
# Add to promtail config
scrape_configs:
  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          __path__: /var/log/syslog
```

### Dedicated Syslog Server

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: syslog-server
  namespace: infrastructure
spec:
  replicas: 1
  selector:
    matchLabels:
      app: syslog
  template:
    metadata:
      labels:
        app: syslog
    spec:
      containers:
      - name: rsyslog
        image: rsyslog/syslog_appliance_alpine:latest
        ports:
        - containerPort: 514
          protocol: UDP
        - containerPort: 514
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: syslog
  namespace: infrastructure
spec:
  type: NodePort
  selector:
    app: syslog
  ports:
  - name: syslog-udp
    port: 514
    targetPort: 514
    protocol: UDP
    nodePort: 30514
  - name: syslog-tcp
    port: 514
    targetPort: 514
    protocol: TCP
    nodePort: 30515
```

## Kerberos/FreeIPA (Optional)

### Purpose

Enterprise identity management for:
- Single Sign-On (SSO)
- Samba shares
- Wi-Fi authentication (RADIUS)

### Deployment Options

#### Option 1: FreeIPA in Kubernetes

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: freeipa
  namespace: idm
spec:
  serviceName: freeipa
  replicas: 1
  selector:
    matchLabels:
      app: freeipa
  template:
    metadata:
      labels:
        app: freeipa
    spec:
      hostNetwork: true
      nodeSelector:
        kubernetes.io/hostname: masternode
      containers:
      - name: freeipa
        image: freeipa/freeipa-server:rocky-9
        args:
          - ipa-server-install
          - --realm=VMSTATION.LOCAL
          - --domain=vmstation.local
          - --setup-dns
          - --no-forwarders
          - -U
        ports:
        - containerPort: 88   # Kerberos
        - containerPort: 464  # Kpasswd
        - containerPort: 389  # LDAP
        - containerPort: 636  # LDAPS
        volumeMounts:
        - name: freeipa-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: freeipa-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: local-path
      resources:
        requests:
          storage: 10Gi
```

#### Option 2: Dedicated VM

For production, consider deploying FreeIPA on a dedicated VM:

1. Install Rocky Linux 9 or AlmaLinux 9
2. Install FreeIPA: `dnf install freeipa-server`
3. Run installer: `ipa-server-install`
4. Configure clients to join realm

#### Option 3: OIDC Integration

Use external identity providers:
- Keycloak
- Dex
- Auth0

### Security Considerations

- Treat KDC as high-value asset
- Isolate network access
- Use strong secrets
- Regular backups
- NetworkPolicy isolation

## Verify Infrastructure

### Check All Components

```bash
kubectl get pods -n infrastructure
```

### Check NTP

```bash
kubectl get pods -n infrastructure -l app=chrony-ntp
```

### Check Syslog (if deployed)

```bash
kubectl get pods -n infrastructure -l app=syslog
```

## Troubleshooting

### Chrony Not Syncing

```bash
# Check chrony logs
kubectl logs -n infrastructure -l app=chrony-ntp

# Check capabilities
kubectl get pods -n infrastructure -l app=chrony-ntp -o yaml | grep -A5 capabilities
```

### Syslog Not Receiving

```bash
# Test syslog connectivity
nc -u 192.168.4.63 30514 <<< "test message"

# Check logs
kubectl logs -n infrastructure -l app=syslog
```

### FreeIPA Issues

```bash
# Check FreeIPA logs
kubectl logs -n idm freeipa-0

# Test kerberos
kinit admin@VMSTATION.LOCAL
```

## Related Documentation

- [DNS Infrastructure](../components/infrastructure/dns.md)
- [NTP/Chrony](../components/infrastructure/ntp-chrony.md)
- [Syslog](../components/infrastructure/syslog.md)
- [Kerberos](../components/infrastructure/kerberos.md)
