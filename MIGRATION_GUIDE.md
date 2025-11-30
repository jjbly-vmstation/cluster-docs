# Migration Guide: VMStation Repository Restructuring

## Overview
This guide documents the migration from the legacy VMStation monorepo to the new modular vmstation-org repository structure. It clarifies the new boundaries, directory layout, and the rationale for the split, with a focus on power management and playbook organization.

---

## Power Management Split

- **cluster-setup** now owns all setup and configuration for power management:
  - `power-management/playbooks/` (autosleep, wake handler, spin-down)
  - `systemd/` (autosleep and wake event units)
- **cluster-tools** now owns all operational scripts:
  - `power-management/` (wake-on-lan, power state checks, log collection)

### Rationale
- **Setup/configuration** (systemd, playbooks) belongs in `cluster-setup` for initial deployment and configuration.
- **Operational tools** (scripts for waking, checking, collecting logs) belong in `cluster-tools` for day-to-day operations.

---

## Playbook Consolidation in cluster-config

- All playbooks are now under `ansible/playbooks/`.
- The legacy `playbooks/` directory has been removed.
- Documentation and references have been updated to reflect this change.

---

## Documentation Centralization

- All detailed component documentation has been moved to `cluster-docs/components/`.
- Each repo now only contains its `README.md` and `IMPROVEMENTS_AND_STANDARDS.md`.
- Refer to `cluster-docs/components/` for:
  - Application deployment
  - Infrastructure services
  - Monitoring and troubleshooting
  - Power management
  - Validation, diagnostics, and testing

---

## Best Practices
- Each repo has an `IMPROVEMENTS_AND_STANDARDS.md` outlining best practices and standards.
- All Ansible, shell, and Kubernetes code follows industry standards for idempotency, security, and maintainability.

---

## Summary Table
| Repository                | Scope                                 | Docs Location                  |
|---------------------------|---------------------------------------|-------------------------------|
| cluster-infra             | Cluster provisioning, Kubespray        | cluster-docs/components/      |
| cluster-config            | System config, infrastructure services | cluster-docs/components/      |
| cluster-application-stack | Application manifests                  | cluster-docs/components/      |
| cluster-monitor-stack     | Monitoring stack                       | cluster-docs/components/      |
| cluster-setup             | Bootstrap, auto-sleep setup            | cluster-docs/components/      |
| cluster-tools             | Operational scripts, validation        | cluster-docs/components/      |

---

## Migration Complete
All major migration and cleanup tasks are complete. For further changes, follow the standards in each repo and keep documentation centralized.
