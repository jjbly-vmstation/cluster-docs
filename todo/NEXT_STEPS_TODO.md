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

---

# Unified Storage Strategy for Nextcloud (Future Planning)

This section is for planning unified storage for Nextcloud, to be implemented only after monitoring is fully validated and deployment is stable.

## 💾 Unified Storage Options

### Option 1: Distributed Storage with Rook/Ceph ⭐ Recommended
**How it works:**
- Both nodes contribute their storage to a distributed filesystem
- Nextcloud sees one unified storage pool
- Files automatically replicated for redundancy
- Survives single-node failure

**Setup:**
- Storage contribution:
  - storagenodet3500: ~3TB contributed
  - homelab: ~generous storage contributed
  - Combined into Ceph pool accessible cluster-wide

**Pros:**
- ✅ True unified storage
- ✅ Automatic replication
- ✅ High availability
- ✅ Nextcloud just uses one PVC

**Cons:**
- ⚠️ Requires minimum 3 nodes (you only have 3 total including master)
- ⚠️ Higher complexity
- ⚠️ Storage overhead for replication

**Setup Steps:**
```bash
# Install Rook operator
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.12/deploy/examples/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.12/deploy/examples/operator.yaml

# Create Ceph cluster using both nodes
# cluster-application-stack/manifests/rook-ceph-cluster.yaml
```

### Option 2: MinIO Distributed Storage ⭐⭐ Best Balance
**How it works:**
- MinIO runs on both storage nodes
- Acts as S3-compatible unified storage
- Erasure coding for redundancy
- Nextcloud uses S3 as primary storage

**Pros:**
- ✅ Simple setup
- ✅ Works with 2 nodes
- ✅ S3-compatible (future-proof)
- ✅ Built-in redundancy
- ✅ No complex cluster management

**Cons:**
- ⚠️ Nextcloud needs S3 plugin
- ⚠️ Slightly higher CPU usage

**Setup Steps:**
```yaml
# cluster-application-stack/manifests/minio-distributed.yaml
apiVersion: v1
kind: Service
metadata:
  name: minio
spec:
  ports:
    - port: 9000
      name: api
    - port: 9001
      name: console
  clusterIP: None
  selector:
    app: minio
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: minio
spec:
  serviceName: minio
  replicas: 2
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - minio
              topologyKey: kubernetes.io/hostname
      containers:
        - name: minio
          image: minio/minio:latest
          args:
            - server
            - http://minio-{0...1}.minio.default.svc.cluster.local/data
            - --console-address
            - ":9001"
          env:
            - name: MINIO_ROOT_USER
              value: "admin"
            - name: MINIO_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: minio-secret
                  key: password
          volumeMounts:
            - name: data
              mountPath: /data
      nodeSelector:
        vmstation.io/storage: "true"
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Ti  # Adjust per node
        storageClassName: local-path
```

---

**Decision:**
Choose the option that best fits your node count, redundancy needs, and operational complexity. Implement only after monitoring and deployment are confirmed stable.
