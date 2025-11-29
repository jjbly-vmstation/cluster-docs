# NTP/Chrony

Time synchronization service.

## Overview

Chrony provides cluster-wide time synchronization, critical for:
- Kubernetes certificates
- Log correlation
- Kerberos authentication

## Deployment

Deployed as DaemonSet on all nodes.

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

## Configuration

### NTP Servers

Default servers in chrony.conf:

```
server pool.ntp.org iburst
server time.cloudflare.com iburst
```

### Ansible Configuration

```yaml
# roles/preflight-rhel10/defaults/main.yml
preflight_chrony_servers:
  - pool.ntp.org
  - time.cloudflare.com
```

## Verification

### Check Pod Status

```bash
kubectl get pods -n infrastructure -l app=chrony-ntp
```

### Check Time Sync

On each node:

```bash
chronyc tracking
chronyc sources
```

### Validation Script

```bash
./tests/validate-time-sync.sh
```

## Troubleshooting

### Pod Not Starting

Check capabilities:

```bash
kubectl describe pod -n infrastructure -l app=chrony-ntp
```

SYS_TIME capability required.

### Time Not Syncing

1. Check NTP servers reachable
2. Check firewall (UDP 123)
3. Check chrony status on node

```bash
ssh <node> "chronyc sources"
```

### Clock Drift

```bash
# Check drift
chronyc tracking | grep "System time"

# Force sync
chronyc makestep
```

## Related Documentation

- [Infrastructure Services](../../deployment/infrastructure-services.md)
- [Prerequisites](../../getting-started/prerequisites.md)
