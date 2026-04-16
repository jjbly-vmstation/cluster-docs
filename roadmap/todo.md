This is a massive maturity leap for your repo. Moving from static YAML manifests (or manual `kubectl apply` commands) to **Kustomize** allows you to keep your "Base" configurations clean while using "Overlays" to handle the specific IP addresses and settings for your R430, T3500, and Windows nodes.

I have integrated this **GitOps/Kustomize refactor** into the master plan.

---

```markdown
# Infrastructure Refactor: Kustomize & LoadBalancer Pivot
**Version:** 11.0
**Focus:** Declarative GitOps via Kustomize & Floating VIPs

---

## 1. The Repository Refactor (Kustomize Strategy)
The `cluster-setup` repo will be restructured to separate **Global Bases** from **Node-Specific Overlays**.

### 1.1 New Directory Structure
```text
cluster-setup/
├── base/                   # Golden manifests (Deployment, ClusterIP)
│   ├── nginx-ingress/
│   ├── cert-manager/
│   └── keycloak/
└── overlays/               # Environment-specific tweaks
    ├── production/         # Your main VMSTATION environment
    │   ├── kustomization.yaml
    │   └── patches/
    │       ├── coredns replicating AD controller dns 
    │       ├── metallb-ip-pool.yaml   # Defines .54-.59
    │       └── ingress-lb-patch.yaml  # Forces Ingress to .54

example^

```

### 1.2 The "Patch" Logic
Instead of editing the Nginx manifest directly, we use a Kustomize patch to enforce the LoadBalancer IP:
```yaml
# overlays/production/patches/ingress-lb-patch.yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.4.54
```

---

## 2. Updated Infrastructure Logic

### 2.1 MetalLB & VIP Management
- **Deployment Method:** Kustomize-managed `IPAddressPool`.
- **Primary Range:** `192.168.4.53 - 192.168.4.59`.
- **Refactor Goal:** No app should ever be exposed via `NodePort`. If an app needs external access, it gets a LoadBalancer IP or an Ingress rule pointing to the `.54` VIP.



### 2.2 Network Inventory (v11.0)
| Layer | IP Range | Assignment Logic |
| :--- | :--- | :--- |
| **ISP/Edge** | .1 - .15 | Eero (Gateway), Cisco (L3 Switch) |
| **Infra VIPs** | .53 - .54 | CoreDNS (.53), Nginx Proxy (.54) |
| **Spare VIPs** | .55 - .59 | Future High-Availability services |
| **Nodes** | .60 - .72 | .61: T3500 | .62: R430 (AD) | .63: Masternode |

---

## 3. Deployment Action Plan

1. **Purge "Ghost" Configs:** Run `kubectl delete node homelab` to finish the RHEL -> WS2025 cleanup.
2. **Kustomize Build:** Test the new repo structure locally:
   ```bash
   kubectl kustomize overlays/production | kubectl apply -f -
   ```
3. **Source of Truth Check:** Ensure the R430 (`.62`) DNS Forwarder points to the CoreDNS VIP (`.53`) to resolve Kustomize-deployed services.

---

## 4. Hardware & Persistence (RACADM)
- **R430:** `racadm set BIOS.SysSecurity.AcPwrRcvry On` (Ensures the Kustomize "Source of Truth" controller is always up).
- **Cisco 3650:** Save LACP configurations to support the increased traffic from the new LoadBalancer-based routing.
```

---

### 💡 Why Kustomize is the "Right" Choice here:
Since you are dealing with a **Hybrid Cluster** (Windows + Linux), you can use Kustomize **NodeSelectors** in your overlays. This ensures that Linux-only pods (like Nginx) stay on your Debian nodes, while Windows-compatible pods can be easily patched to run on the `homelab` node without rewriting the whole base manifest.

**Since you're rewriting the repo, do you want a `kustomization.yaml` template that specifically handles the MetalLB IP pool and the Nginx patch together?**