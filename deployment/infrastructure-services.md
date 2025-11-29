# Infrastructure Services

Deploy core infrastructure services.

## Overview

Infrastructure services include:
- **NTP/Chrony** - Time synchronization
- **Syslog** - Centralized logging
- **Kerberos/FreeIPA** - SSO (optional)

## Quick Deployment

```bash
./deploy.sh infrastructure
```

## NTP/Chrony

Time synchronization is critical for Kubernetes certificates and log correlation.

### Deployment

Chrony runs as a DaemonSet on all nodes.

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
    spec:
      hostNetwork: true
      containers:
      - name: chrony
        image: cturra/ntp:latest
        securityContext:
          capabilities:
            add: ["SYS_TIME"]
```

### Verification

```bash
# Check pods
kubectl get pods -n infrastructure -l app=chrony-ntp

# Check time sync on node
chronyc tracking
chronyc sources
```

## Syslog

Syslog collection via Promtail (already deployed with monitoring).

### Alternative: Dedicated Syslog Server

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: syslog-server
  namespace: infrastructure
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: rsyslog
        image: rsyslog/syslog_appliance_alpine:latest
        ports:
        - containerPort: 514
          protocol: UDP
```

### Query Syslog in Grafana

```logql
{job="syslog"}
```

## Kerberos/FreeIPA (Optional)

Enterprise identity management for SSO.

### Status

**Not deployed** - Optional component for advanced setups.

### Purpose

- Single Sign-On (SSO)
- Samba share authentication
- Wi-Fi authentication (via RADIUS)
- Centralized user management

### Deployment Options

**Option 1: FreeIPA in Kubernetes**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: freeipa
  namespace: idm
spec:
  serviceName: freeipa
  replicas: 1
  template:
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
```

**Option 2: Dedicated VM**

For production, run FreeIPA on a dedicated VM.

**Option 3: OIDC**

Use external identity providers (Keycloak, Dex, Auth0).

### Security Considerations

- High-value target - isolate access
- Use strong secrets
- Regular backups
- NetworkPolicy isolation

## DNS (CoreDNS)

CoreDNS provides cluster DNS resolution.

### Check Status

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

### Test Resolution

```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
```

### Fix DNS Issues

If nodelocaldns conflicts:

```bash
kubectl delete daemonset nodelocaldns -n kube-system

# Update kubelet on all nodes
sed -i 's/169.254.25.10/10.233.0.3/g' /var/lib/kubelet/config.yaml
systemctl restart kubelet
```

## Verification

```bash
# Check infrastructure pods
kubectl get pods -n infrastructure

# Check NTP
kubectl get pods -n infrastructure -l app=chrony-ntp

# Validate time sync
./tests/validate-time-sync.sh
```

## Related Documentation

- [NTP/Chrony](../components/infrastructure/ntp-chrony.md)
- [Syslog](../components/infrastructure/syslog.md)
- [Kerberos](../components/infrastructure/kerberos.md)
- [DNS](../components/infrastructure/dns.md)
