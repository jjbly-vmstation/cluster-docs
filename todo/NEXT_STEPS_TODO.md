# VMStation Migration: Next Steps TODO Checklist

This file tracks the prioritized migration and improvement actions for the VMStation modular cluster project. Each step includes references to guiding documents for implementation and planning.

---

## TODO Checklist with Guiding Documents

### 1. Migrate deploy.sh to cluster-setup
- Move and refactor the deploy.sh orchestration script from the monorepo to cluster-setup/orchestration/deploy.sh. This is the baseline for all cluster operations.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/DEPLOYMENT_WORKFLOW.md
  - cluster-docs/reference/INVENTORY_MANAGEMENT.md

### 2. Migrate Kubespray to cluster-infra
- Move Kubespray integration scripts and roles from the monorepo to cluster-infra/scripts and cluster-infra/ansible/roles. Add Kubespray as a submodule or subtree.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/DEPLOYMENT_WORKFLOW.md
  - cluster-docs/reference/INVENTORY_MANAGEMENT.md

### 3. Create unified inventory in cluster-infra
- Establish cluster-infra/inventory/production/hosts.yml as the single source of truth for inventory. Remove duplicates and update all playbooks to reference this inventory.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/reference/INVENTORY_MANAGEMENT.md

### 4. Document multi-repo deployment workflow
- Create cluster-docs/getting-started/DEPLOYMENT_WORKFLOW.md to document the end-to-end deployment process across all repos.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/DEPLOYMENT_WORKFLOW.md

### 5. Migrate operational tools to cluster-tools
- Populate cluster-tools with validation, diagnostic, remediation, power management scripts, and the BATS test framework from the monorepo.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/operations/CROSS_REPO_OPERATIONS.md
  - cluster-docs/operations/ROLLBACK_PROCEDURES.md

### 6. Add Ansible automation to cluster-monitor-stack
- Migrate monitoring deployment playbooks and operational scripts from the monorepo to cluster-monitor-stack/ansible/playbooks.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/operations/MONITORING_VM_METRICS.md
  - cluster-docs/operations/CROSS_REPO_OPERATIONS.md

### 7. Add Ansible automation to cluster-application-stack
- Create deployment playbooks for applications, including Jellyfin, in cluster-application-stack/ansible/playbooks and roles.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/MULTI_REPO_DEPLOYMENT.md
  - cluster-docs/operations/CROSS_REPO_OPERATIONS.md

### 8. Create migration verification tools
- Build scripts in cluster-tools/migration to compare monorepo and modular repos, reporting missing files/functionality.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/migration/VERIFICATION_GUIDE.md
  - cluster-docs/migration/MIGRATION_ROADMAP.md

### 9. Complete Terraform modules in cluster-infra
- Develop production-ready Terraform modules for Proxmox, VMware, network, and storage in cluster-infra/terraform.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/DEPLOYMENT_WORKFLOW.md
  - cluster-docs/getting-started/MULTI_REPO_DEPLOYMENT.md

### 10. Add CI/CD pipeline definitions to cluster-setup
- Create .drone.yml and GitHub Actions workflows for automated deployment, validation, and rollback in cluster-setup.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/DEPLOYMENT_WORKFLOW.md
  - cluster-docs/getting-started/MULTI_REPO_DEPLOYMENT.md

### 11. Implement rollback capabilities
- Add state snapshot and rollback scripts to cluster-setup/orchestration for safe recovery from failed deployments.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/operations/ROLLBACK_PROCEDURES.md

### 12. Add automated drift detection
- Implement scripts in cluster-tools/drift-detection to detect inventory, config, and manifest drift across repos.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/operations/CROSS_REPO_OPERATIONS.md

### 13. Create Helm charts for applications
- Develop Helm charts for application deployments in cluster-application-stack/helm-charts.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/MULTI_REPO_DEPLOYMENT.md

### 14. VM setup and metrics integration
- Finalize VM provisioning and monitoring integration, referencing hypervisor-level exporters and Prometheus configuration.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/operations/MONITORING_VM_METRICS.md

### 15. Archive legacy monorepo
- Deprecate and archive the vmstation monorepo after migration is complete. Add deprecation notice and update documentation.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/migration/MIGRATION_ROADMAP.md

### 16. Set up cross-repo CI/CD
- Configure CI/CD to test and validate multi-repo workflows and integration.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/MULTI_REPO_DEPLOYMENT.md

### 17. External documentation updates
- Update all external links and documentation to reference the new modular repos and workflows.
**Guiding Documents:**
  - MIGRATION_ANALYSIS.md
  - cluster-docs/getting-started/MULTI_REPO_DEPLOYMENT.md
  - cluster-docs/migration/MIGRATION_ROADMAP.md

---

Update this file as you complete each step. Reference the guiding documents for details, implementation patterns, and migration context.
