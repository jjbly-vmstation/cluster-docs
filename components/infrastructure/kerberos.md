# Kerberos / FreeIPA

Single Sign-On and identity management.

## Status

**Not Deployed** - Optional component for advanced setups.

## Purpose

- Single Sign-On (SSO)
- Samba share authentication
- Wi-Fi authentication (via RADIUS)
- Centralized user management

## Deployment Options

### Option 1: FreeIPA in Kubernetes

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

### Option 2: Dedicated VM

For production, run FreeIPA on a dedicated VM.

### Option 3: OIDC Alternative

Use external identity providers:
- Keycloak
- Dex
- Auth0

## Prerequisites

### Time Sync

Kerberos is sensitive to clock drift:

```bash
# Verify chrony running
chronyc tracking
```

### DNS

FreeIPA requires DNS:
- Either internal DNS
- Or configure existing DNS

### Storage

Persistent volume required:
- `/var/lib/freeipa` - IPA data
- `/var/lib/dirsrv` - LDAP data

## Configuration

### Realm Setup

```bash
ipa-server-install \
  --realm=VMSTATION.LOCAL \
  --domain=vmstation.local \
  --setup-dns \
  --no-forwarders
```

### Add User

```bash
ipa user-add testuser \
  --first=Test \
  --last=User \
  --password
```

### Create Service Principal

```bash
ipa service-add host/storagenodet3500.vmstation.local
ipa-getkeytab -s freeipa.vmstation.local \
  -p host/storagenodet3500.vmstation.local \
  -k /etc/krb5.keytab
```

## Client Configuration

### Install IPA Client

```bash
ipa-client-install \
  --server=freeipa.vmstation.local \
  --domain=vmstation.local \
  --realm=VMSTATION.LOCAL
```

### Verify

```bash
kinit admin@VMSTATION.LOCAL
klist
```

## Security Considerations

- High-value target - isolate access
- Use NetworkPolicy
- Strong admin password
- Regular backups
- Monitor for unauthorized access

## Related Documentation

- [Infrastructure Services](../../deployment/infrastructure-services.md)
- [NTP/Chrony](ntp-chrony.md)
- [DNS](dns.md)
