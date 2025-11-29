# Kerberos

Optional enterprise identity management for VMStation.

## Status

**Not deployed** - Optional component for SSO and enterprise authentication.

## Purpose

Kerberos/FreeIPA enables:
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

For production:
1. Install Rocky Linux 9
2. Install FreeIPA: `dnf install freeipa-server`
3. Run installer
4. Configure clients

### Option 3: OIDC

Use external identity providers:
- Keycloak
- Dex
- Auth0

## Prerequisites

- Time sync (Chrony) ✅
- DNS ✅
- Persistent storage ✅
- Network isolation

## Security Considerations

- High-value target - isolate access
- Use strong secrets
- Regular backups
- NetworkPolicy isolation

## Configuration Steps

1. Deploy FreeIPA StatefulSet
2. Initialize realm
3. Create service principals
4. Configure clients
5. Test authentication

## Verification

```bash
# Test kerberos
kinit admin@VMSTATION.LOCAL

# List principals
klist
```

## Related Documentation

- [Infrastructure Services](../../deployment/infrastructure-services.md)
- [Architecture Overview](../../architecture/overview.md)
