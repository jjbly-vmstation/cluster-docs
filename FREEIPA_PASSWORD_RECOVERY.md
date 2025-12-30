## FreeIPA Directory Manager and Admin Password Recovery

This document describes how to reset the Directory Manager (DM) password and recover/reset the FreeIPA admin password inside a FreeIPA pod (Kubernetes deployment).

### 1. Get a shell inside the FreeIPA pod as root
```
kubectl exec -n identity -it freeipa-0 -- bash
```

### 2. Stop FreeIPA services
```
ipactl stop
```

### 3. Find your Directory Server instance name
```
ls /etc/dirsrv/
# Look for a directory like slapd-<INSTANCENAME>
# Example: slapd-VMSTATION-LOCAL
```

### 4. Reset the Directory Manager password
```
dsconf <INSTANCENAME> directory_manager password_change
# Example:
dsconf VMSTATION-LOCAL directory_manager password_change
```
You will be prompted to enter a new Directory Manager password.

### 5. Start FreeIPA services
```
ipactl start
```

### 6. Reset the FreeIPA admin password using the new Directory Manager password
```
ldappasswd -H ldap://localhost -x -ZZ -D "cn=Directory Manager" -W -S "uid=admin,cn=users,cn=accounts,dc=vmstation,dc=local"
```
You will be prompted for:
- New password (for admin)
- Re-enter new password
- LDAP Password (enter the new Directory Manager password)

If you see `Result: Confidentiality required (13)`, always use the `-ZZ` flag for StartTLS.

---
**Summary:**
- Use `dsconf <INSTANCENAME> directory_manager password_change` to reset Directory Manager password.
- Use `ldappasswd -H ldap://localhost -x -ZZ -D "cn=Directory Manager" -W -S "uid=admin,cn=users,cn=accounts,dc=vmstation,dc=local"` to reset the FreeIPA admin password.

Keep this document for future FreeIPA password recovery operations.