# NTP/Chrony

Chrony provides time synchronization across the VMStation cluster.

## Overview

| Property | Value |
|----------|-------|
| Service | Chrony |
| Deployment | DaemonSet |
| Namespace | infrastructure |

## Purpose

Time synchronization is critical for:
- Kubernetes certificate validity
- Log correlation
- Monitoring accuracy
- Kerberos authentication

## Configuration

### DaemonSet

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

### Chrony Config

```
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst

driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

## Verification

### Check Pods

```bash
kubectl get pods -n infrastructure -l app=chrony-ntp
```

### Check Sync Status

```bash
# On each node
chronyc tracking
chronyc sources
```

### Validation Script

```bash
./tests/validate-time-sync.sh
```

## Troubleshooting

### Chrony Not Syncing

```bash
# Check pod logs
kubectl logs -n infrastructure -l app=chrony-ntp

# Check on node
chronyc sources -v
```

### Time Drift

```bash
# Force sync
chronyc makestep
```

## Related Documentation

- [Infrastructure Services](../../deployment/infrastructure-services.md)
- [Prerequisites](../../getting-started/prerequisites.md)
