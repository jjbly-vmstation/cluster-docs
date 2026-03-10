
---

# Project Migration: RHEL to Windows Server 2025

**Role:** Identity & Infrastructure Architect

**Environment:** Dell PowerEdge R430 (Dual Xeon E5-2660 v4, 128GB RAM, RAID 10)

## 1. Executive Summary

This document outlines the strategic migration of the primary lab hypervisor and identity core from Red Hat Enterprise Linux (RHEL) to Windows Server 2025. The transition was driven by the requirement for an integrated, enterprise-grade Identity Provider (IdP) and the need for native support of modern Windows security features (vTPM, Secure Boot) within a virtualized Windows 11 Enterprise environment.

---

## 2. Technical Reasoning for Migration

### A. Centralized Identity Management (AD DS vs. LDAP)

While RHEL offers robust LDAP and FreeIPA solutions, **Active Directory Domain Services (AD DS)** remains the industry standard for enterprise identity.

* **Reasoning:** Implementing AD DS on Windows Server 2025 provides native Group Policy Object (GPO) management for Windows 11 endpoints, which is critical for the "Dad-VM" use case (automated mapping of network drives and browser configurations).

### B. Enterprise PKI Integration (AD CS)

* **Reasoning:** The migration allowed for the deployment of **Active Directory Certificate Services (AD CS)**. By establishing a local Enterprise Root CA, the lab now supports automated certificate enrollment via SCEP/NDES for network infrastructure, such as the Cisco 3650v2 switch, ensuring encrypted management traffic.

### C. Hyper-V vs. KVM/Libvirt

* **Reasoning:** Windows Server 2025 Hyper-V provides superior integration for Windows 11 Enterprise guests, specifically regarding **Enhanced Session Mode** (RDP over VMBus) and native vTPM passthrough. This ensures that the virtualized desktop environment is indistinguishable from bare metal for the end-user.

### D. Federated Identity (Keycloak Integration)

* **Reasoning:** Using Windows Server as the "Source of Truth" allowed for the implementation of **LDAP User Federation** into Keycloak. This architecture demonstrates a hybrid identity model where legacy AD credentials authorize modern OIDC/SAML applications.

---

## 3. Migration Roadmap & Execution

### Phase 1: Storage Preservation

* Configured the PERC H730P Mini controller for **RAID 10** to balance high-IOPS performance and redundancy.
* Established a cross-platform storage bridge by mounting a Linux-based **NFS ISO repository** onto the Windows host via the "Services for NFS" client.

### Phase 2: Core Services Deployment

1. **OS Provisioning:** Clean installation of Windows Server 2025 (Datacenter Edition).
2. **Role Installation:** Deployed AD DS, DNS, and AD CS.
3. **Forest Logic:** Created the `vmstation.local` forest root, utilizing the 2025 functional level to leverage modern Kerberos enhancements.

### Phase 3: Workload Migration

* Provisioned a Windows 11 Enterprise VM using **Packer-automated VHDX artifacts**.
* Configured **NFS anonymous UID/GID mapping** (Registry Tuning) to ensure the hypervisor maintained non-interactive access to centralized media stores.

---

## 4. Key Challenges & Resolutions

| Challenge | Resolution |
| --- | --- |
| **NFS Network Error 53** | Identified a protocol mismatch; enabled **NFS v3** and **insecure** flags on the Linux exporter and aligned Windows registry (AnonymousUid/Gid) to map to Root (0). |
| **AD CS Blocker** | Resolved a DC promotion failure by uninstalling the standalone CA role and re-installing it as an **Enterprise CA** post-promotion. |
| **Firewall Inoperability** | Leveraged PowerShell to inject RPC and NFS exceptions into the **DomainAuthenticated** firewall profile. |

---

## 5. Conclusion

The migration has successfully transitioned the lab from a series of isolated Linux services to a cohesive, domain-joined ecosystem. The current architecture provides a professional-grade testing ground for Group Policy orchestration, PKI management, and federated identity, significantly increasing the production-readiness of the environment.

---
