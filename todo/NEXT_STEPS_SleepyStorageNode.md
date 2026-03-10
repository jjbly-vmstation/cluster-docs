# Homelab Project: T3500 Power Management & Eero Bypass Architecture

## Objective

Minimize the idle power consumption of the Dell PowerEdge T3500 storage/Jellyfin node (dual 2011 Xeons) by implementing a smart sleep/wake cycle. This architecture bypasses the Eero 6 mesh router's DNS hijacking and broadcast dropping behavior by utilizing the Masternode and the Cisco Catalyst 3650 v2 switch to handle Layer 2 Wake-on-LAN (WoL) routing.

---

## Phase 1: The "Smart Sleep" Implementation

**Target Node:** `storagenodet3500`

### Problem

Standard `cron` jobs trigger blindly based on the clock, causing the server to immediately go back to sleep if it is woken up just before a scheduled cron interval.

### Solution

A bash script that evaluates the system's state before executing `systemctl suspend`.

**1. Create the Script**
Create `/usr/local/bin/smart-sleep.sh`:

```bash
#!/bin/bash

# 1. Grace Period: Ensure server stays awake for at least 15 minutes (900 seconds) after booting/waking
UPTIME=$(cut -d. -f1 /proc/uptime)
if [ "$UPTIME" -lt 900 ]; then
    exit 0
fi

# 2. Connection Check: Look for active TCP connections on critical ports
# 8096 = Jellyfin | 445 = SMB/Storage | 22 = SSH
ACTIVE_CONNS=$(ss -tn state established \'( dport = :8096 or sport = :8096 or dport = :445 or sport = :445 or dport = :22 or sport = :22 )\' | wc -l)

# 'ss' outputs a header row; a line count > 1 means active connections exist
if [ "$ACTIVE_CONNS" -gt 1 ]; then
    exit 0
fi

# 3. Action: If uptime > 15 mins AND no active connections exist, suspend
systemctl suspend

```

**2. Make Executable**

```bash
chmod +x /usr/local/bin/smart-sleep.sh

```

**3. Schedule via Cron**
Run the evaluation every 5 minutes.

```bash
crontab -e
# Add the following line:
*/5 * * * * root /usr/local/bin/smart-sleep.sh

```

---

## Phase 2: The Masternode WoL Bridge

**Target Node:** `masternode` (15-25W Minipc)

### Problem

The Eero 6 drops Layer 2 WoL broadcast packets and refuses to route CoreDNS queries properly, breaking standard local network wake setups.

### Solution

Use the always-on masternode as a reverse proxy that listens for Jellyfin traffic, issues the WoL Magic Packet over the Cisco Catalyst switch, and proxies the connection.

**1. Client Configuration (The Bypass)**

* Reconfigure all household clients (TVs, phones) to connect to Jellyfin using the **Masternode's IP address** instead of the T3500's IP or hostname.
* Example: `http://<masternode-ip>:8096`

**2. Deploy WoL Reverse Proxy**

* Deploy a Wake-on-LAN proxy container (e.g., `lazytainer`, `wol-proxy`, or a custom Traefik/Nginx configuration with a WoL trigger script) to the masternode.
* **Proxy Configuration Requirements:**
* **Listen Port:** `8096`
* **Target MAC Address:** `<T3500-MAC-Address>`
* **Target IP Address:** `<T3500-IP>:8096`
* **Timeout/Wait:** Must be configured to hold the HTTP request for 15-20 seconds to allow the dual Xeons to POST and the OS to initialize.



### Traffic Flow Summary

1. TV requests Jellyfin -> hits Masternode (via Eero WiFi, bypassing DNS).
2. Masternode proxy holds request -> sends Magic Packet down the Cisco 3650 switch.
3. T3500 receives L2 broadcast -> wakes up.
4. Proxy detects T3500 is online -> forwards the Jellyfin HTTP request.
5. TV streams video.
6. TV stops streaming -> Smart Sleep script waits 15 minutes -> suspends T3500.

---

**Next Steps / Todo:**

* [ ] Write Ansible playbook to deploy `smart-sleep.sh` and configure cron on the T3500.
* [ ] Select and deploy the WoL reverse proxy container to the masternode.
* [ ] Re-point home devices to the masternode IP.


Here are the two foundational scripts you will need for your to-do list to make this architecture work.

Since you are already heavily utilizing Ansible for your cluster operations, the first is an Ansible playbook to automate the deployment of the Smart Sleep script to your Dell tower. The second is a lightweight Docker Compose stack for your 15W masternode to handle the Wake-on-LAN routing.

### 1. Ansible Playbook: Deploy "Smart Sleep" to the T3500

Save this on your masternode as `deploy-smart-sleep.yml`. It handles creating the script, setting the execution permissions, and registering the 5-minute cron job on your storage node.

```yaml
---
- name: Deploy Smart Sleep Architecture to Storage Node
  hosts: storagenodet3500
  become: true
  tasks:
    - name: Create the Smart Sleep bash script
      copy:
        dest: /usr/local/bin/smart-sleep.sh
        mode: '0755'
        owner: root
        group: root
        content: |
          #!/bin/bash
          # 1. Grace Period: 15 minutes (900 seconds)
          UPTIME=$(cut -d. -f1 /proc/uptime)
          if [ "$UPTIME" -lt 900 ]; then
              exit 0
          fi

          # 2. Connection Check: Jellyfin (8096), SMB (445), SSH (22)
          ACTIVE_CONNS=$(ss -tn state established \'( dport = :8096 or sport = :8096 or dport = :445 or sport = :445 or dport = :22 or sport = :22 )\' | wc -l)

          if [ "$ACTIVE_CONNS" -gt 1 ]; then
              exit 0
          fi

          # 3. Sleep if idle
          systemctl suspend

    - name: Schedule Smart Sleep via Cron every 5 minutes
      cron:
        name: "Evaluate Smart Sleep"
        minute: "*/5"
        job: "/usr/local/bin/smart-sleep.sh"
        user: root

```

### 2. Docker Compose: The Masternode WoL Proxy

Writing a custom reverse proxy from scratch to handle Layer 2 magic packets is a pain. Instead, you can use **Lazytainer**, an open-source container specifically designed to listen for traffic, wake up a sleeping target, and proxy the connection once the target is online.

Run this on your masternode. It intercepts the traffic hitting the Eero WiFi, fires the WoL packet down the Cisco switch, and waits for the dual Xeons to POST before passing the Jellyfin traffic through.

```yaml
# docker-compose.yml for Masternode
version: "3"
services:
  lazytainer-jellyfin:
    image: ghcr.io/vmorganp/lazytainer:latest
    container_name: lazytainer-jellyfin
    environment:
      - PORT=8096                 # The port Lazytainer listens on
      - TARGET_IP=192.168.x.x     # The IP of your T3500
      - TARGET_MAC=XX:XX:XX:XX:XX:XX # The MAC address of the T3500
      - TIMEOUT=15                # How many seconds to wait for the T3500 to wake
      - PROXY_MODE=true           # Proxy the traffic instead of just waking
    ports:
      - "8096:8096"
    restart: unless-stopped

```

*Note: Make sure you update `TARGET_IP` and `TARGET_MAC` with the actual details of your T3500's network interface.*

---

With these two files banked for later, you can tackle that power management project whenever you have the free time.

Would you like me to draft the `server.pkr.hcl` and the required Windows Server `.j2` answer file so you can knock out that final Golden Image today?


That 8GB of RAM on your masternode is definitely the "danger zone" when running the full observability stack (Prometheus/Loki/Grafana) alongside CoreDNS and a K8s control plane. Moving to **containerd** was a smart play—it shaves off a significant amount of overhead compared to the full Docker daemon.

Since you're managing a memory-constrained cluster, here is a breakdown of how to handle the `storagenodet3500` and the `masternode` without OOM (Out Of Memory) crashing them.

---

## 1. Kubernetes Resource Taints & Affinity

With such a massive disparity between your nodes (8GB vs 64GB), you shouldn't let Kubernetes decide where to put things "fairly." You need to force the heavy lifting onto the 64GB RHEL 10 host.

* **Taint the T3500:** Since the T3500 is only for storage/Jellyfin and has very little RAM, you should taint it so only specific pods (like the Jellyfin pod or a CSI storage driver) can run there.
```bash
kubectl taint nodes storagenodet3500 storage-only=true:NoSchedule

```


* **Limit Loki/Prometheus:** Prometheus and Loki are "RAM hungry" by nature because they keep indexes in memory. Ensure you have `resources: limits:` set in your Helm charts or manifests, otherwise, Prometheus will eventually eat all 8GB of your masternode and lock you out of SSH.

---

## 2. Wake-on-LAN for Kubernetes (The "Kube-WoL" approach)

Since you are using Kubernetes, the "Lazytainer" Docker approach I mentioned earlier needs to be adapted for your K8s environment.

Because the T3500 is a K8s node, when it's asleep, Kubernetes will mark it as `NotReady`. To wake it up when someone wants to watch Jellyfin, you can use an **Ingress-based trigger** on your masternode.

### The To-Do List Entry: Kubernetes WoL Controller

Instead of a simple Docker container, you can look into a **K8s Custom Controller** or a simple "Proxy Pod" on the masternode that:

1. Watches for traffic on the Jellyfin Ingress.
2. Executes a `wakeonlan` command from within a pod if the T3500 node is `NotReady`.
3. Utilizes the Cisco 3650 switch to broadcast the packet.

---

## 3. The Windows Server Edition Build

Since we’ve mastered the Windows 11 `autounattend.xml`, let’s pivot to your Windows Server build. Windows Server is actually much more stable in a 6GB/8GB environment if you use **Server Core** (no GUI).

### Your Checklist for the Server VM:

* **Target Image:** Ensure you use `Windows Server 2022 SERVERDATACENTER` (or Standard) in the XML.
* **RAM Allocation:** You can safely drop the Packer `memory` variable to `4096` or `6144` for the build process, which will keep your homelab host from feeling the squeeze if you're running multiple builds.
* **Storage Adapter:** Use `lsilogic-sas` or `sata`. Unlike Win11, Server doesn't throw a fit without NVMe.

---

### Project Update: The "Next Step"

Since the T3500 is power-hungry and memory-light, would you like me to write a **Kubernetes Job** or **CronJob** manifest that periodically checks if the T3500 is idle and uses the Kubernetes API to "drain" the node before sending it to sleep?

This would ensure that no important "analytics" pods are accidentally killed when the T3500 shuts down to save power.