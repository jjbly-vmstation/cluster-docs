# Project: VMSTATION Full-Stack Infrastructure Refactor
**Version:** 10.0 (Finalized IP Schema & Infrastructure Range)
**Date:** April 2026
**Primary DC:** R430 (192.168.4.62)

---

## 1. Network Segmentation & IP Schema

| Range | Purpose | Devices |
| :--- | :--- | :--- |
| **.1 – .15** | **Network Core** | Eero 6, Cisco 3650 Switch, Management IPs |
| **.33 – .52** | **Available Space** | Future growth / Secondary services |
| **.53** | **K8s CoreDNS** | MetalLB LoadBalancer (Fixed) |
| **.54 – .59** | **K8s Infra** | **.54: Nginx Reverse Proxy** |
| **.60 – .72** | **K8s Compute** | .61: T3500 \| .62: R430 (AD) \| .63: Masternode |

---

## 2. Gateway & ISP Integration (Oxio/Eero)

### 2.1 DHCP & NAT Logic
- **Eero 6:** Remains the DHCP Server + NAT Gateway (Required for Oxio connectivity).
- **DNS Hand-off:** Eero hands out **192.168.4.62** (R430) to all clients.

---

## 3. Identity & DNS "Source of Truth"

### 3.1 Active Directory (R430 @ .62)
- **Primary DNS:** AD handles all `.vmstation.local` requests.
- **Forwarder:** Points to **192.168.4.53** (Cluster CoreDNS) for internal service discovery.
- **Upstream:** Points to `1.1.1.1` for internet resolution.

### 3.2 Keycloak User Federation
- **LDAP Provider:** Windows AD on R430.
- **IDP Goal:** Keycloak acts as the frontend OIDC/SAML provider, authenticating against the R430 AD database.

---

## 4. Kubernetes Ingress & Proxy Logic

### 4.1 Nginx LoadBalancer (MetalLB)
The Reverse Proxy is promoted from `NodePort` to `Type: LoadBalancer`.
- **Assigned IP:** `192.168.4.54`
- **DNS Mapping:**
    - `*.vmstation.local` —> `192.168.4.54`
    - `sso.vmstation.local` —> `192.168.4.54`
    - `cloud.vmstation.local` —> `192.168.4.54`

### 4.2 Reverse Lookup Zone
Every IP in the **.60 – .72** range must have a manual PTR record in the R430 DNS Manager to ensure Keycloak can perform host verification without timeouts.

---

## 5. Maintenance & Resilience (RACADM)
- **R430 (AD/DNS/Worker):** `racadm set BIOS.SysSecurity.AcPwrRcvry On`
- **Node Recovery:** `kubectl delete node homelab` (Clears RHEL metadata before WS2025 re-join).
- **Cisco 3650:** L3 routing enabled; LACP configured for R430/T3500 storage backbone.