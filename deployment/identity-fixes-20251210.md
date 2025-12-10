```markdown
# Identity Stack Fixes (2025-12-10)

**Date**: 2025-12-10

This document summarizes fixes applied to the identity deployment playbook and manifests on 2025-12-10. The changes address PostgreSQL and Keycloak readiness issues, PersistentVolume binding problems, CA/ClusterIssuer format, and privilege escalation bugs when running the playbook from a non-privileged user.

## Status

- **Status**: ✅ WORKING — PostgreSQL, Keycloak, FreeIPA, ClusterIssuer functional

## Key Problems Addressed

1. PostgreSQL deployment failed when using the Helm subchart (Bitnami paths/images mismatch).
2. PersistentVolumes in `Released` state could not bind to new PVCs.
3. Keycloak fell back to H2 due to missing DB environment variables.
4. Node scheduling for Keycloak and cert-manager components was incorrect.
5. Cert-manager ClusterIssuer failed because the CA secret used `ca.crt` instead of `tls.crt`/`tls.key`.
6. The playbook failed under non-privileged users because many tasks required privilege escalation.

## High-level Solutions

- Deploy PostgreSQL as a standalone StatefulSet (`manifests/identity/postgresql-statefulset.yaml`) and disable the Helm subchart's PostgreSQL via `--set postgresql.enabled=false`.
- Clear stale `spec.claimRef` on PVs that are in `Released` state so they can be re-bound.
- Patch the Keycloak StatefulSet after Helm install to inject DB environment variables (DB_VENDOR, DB_ADDR, DB_PORT, DB_DATABASE, DB_USER, DB_PASSWORD).
- Patch Keycloak StatefulSet to set `nodeSelector` to the masternode so identity components run on the control-plane.
- Create the CA secret using keys `tls.crt` and `tls.key` so cert-manager accepts the CA and ClusterIssuer becomes Ready.
- Add `become: true` to tasks that require root privileges (kubectl, helm, file operations) and create the backup directory up-front with secure permissions.

## Files Added / Modified

- Added: `manifests/identity/postgresql-statefulset.yaml`
- Modified: `ansible/playbooks/identity-deploy-and-handover.yml` (privilege fixes, PV claim clearing, standalone Postgres, Keycloak patches, diagnostics)
- Patch file: `diff-patches/20251210-identity-stack-complete-fix.patch`

## Operational Notes & Verification

Run the playbook as documented (from the repo root):

```bash
cd /opt/vmstation-org/cluster-infra
sudo ansible-playbook -i /opt/vmstation-org/cluster-setup/ansible/inventory/hosts.yml ansible/playbooks/identity-deploy-and-handover.yml --become
```

Verify pods and ClusterIssuer:

```bash
sudo kubectl get pods -n identity -o wide
sudo kubectl get clusterissuer freeipa-ca-issuer
```

If PVs show `Released`, the playbook now includes steps to clear `spec.claimRef` for affected PVs. If you made manual changes to PVs previously, confirm they are reusable before running a fresh deployment.

## Diagnostics and Backup

- The playbook now proactively creates `/root/identity-backup` with secure permissions and stores diagnostics (pod logs, describes, events) there on failures.
- Diagnostic files are timestamped and owned by root with restrictive file modes.

## Next Steps & Recommendations

1. Move manual patches into Helm values or generated manifests where practical.
2. Improve automatic chown handling for PostgreSQL hostPath (current Job may require manual ownership fixes on some systems).
3. Add nodeSelector adjustments for cert-manager components to keep control-plane workloads consolidated.
4. Replace `CHANGEME` placeholders (DB passwords, Keycloak admin) before production.

## References

- `/opt/vmstation-org/diff-patches/20251210-identity-deploy-privilege-fix.patch`
- `/opt/vmstation-org/diff-patches/20251210-identity-stack-complete-fix.patch`
- `/opt/vmstation-org/diff-patches/20251210-IDENTITY-FIXES-SUMMARY.md`

---

*Document generated from diff-patches collected on 2025-12-10.*
```