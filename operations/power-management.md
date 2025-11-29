# Power Management

This guide covers VMStation's power management features including auto-sleep and Wake-on-LAN.

## Overview

VMStation implements energy-efficient operations:
- **Auto-sleep**: Worker nodes sleep after idle period
- **Wake-on-LAN**: Remote wake for sleeping nodes
- **masternode**: Always-on for availability

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    Power Management Flow                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    masternode (Always-on)                    │ │
│  │                                                              │ │
│  │  ┌──────────────────┐  ┌──────────────────────────────────┐ │ │
│  │  │  Auto-sleep      │  │  Wake-on-LAN                     │ │ │
│  │  │  Monitor         │  │  Script                          │ │ │
│  │  │  (hourly check)  │  │  (magic packet)                  │ │ │
│  │  └──────────────────┘  └──────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                          │                │                       │
│              Sleep       │                │ Wake                  │
│              Command     │                │ Packet                │
│                          ▼                ▼                       │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                     Worker Nodes                             │ │
│  │                                                              │ │
│  │  ┌────────────────────┐    ┌────────────────────┐          │ │
│  │  │ storagenodet3500   │    │ homelab            │          │ │
│  │  │ (auto-sleep)       │    │ (auto-sleep)       │          │ │
│  │  └────────────────────┘    └────────────────────┘          │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## Setup Auto-Sleep

### Deploy Auto-Sleep

```bash
./deploy.sh setup
```

### What It Configures

1. **Monitoring script**: `/usr/local/bin/vmstation-autosleep-monitor.sh`
2. **Sleep script**: `/usr/local/bin/vmstation-sleep.sh`
3. **Systemd timer**: `vmstation-autosleep.timer`

### Configuration

Default settings:
- Idle threshold: 2 hours
- Check interval: 15 minutes
- Sleep command: `systemctl suspend`

## Wake-on-LAN

### MAC Addresses

| Node | MAC Address |
|------|-------------|
| storagenodet3500 | b8:ac:6f:7e:6c:9d |
| homelab | d0:94:66:30:d6:63 |

### Manual Wake

```bash
# Wake storagenodet3500
wakeonlan b8:ac:6f:7e:6c:9d

# Wake homelab
wakeonlan d0:94:66:30:d6:63

# Wake all workers
wakeonlan b8:ac:6f:7e:6c:9d d0:94:66:30:d6:63
```

### With Broadcast Address

```bash
wakeonlan -i 192.168.4.255 b8:ac:6f:7e:6c:9d
```

### Install wakeonlan

```bash
# Debian/Ubuntu
sudo apt install wakeonlan

# RHEL/Fedora
sudo dnf install wol
```

## Auto-Sleep Monitor

### How It Works

1. Check for active workloads
2. Check CPU usage
3. Check Jellyfin activity
4. If idle > threshold → sleep

### Monitored Conditions

```bash
# Active pods on worker
kubectl get pods -o wide | grep <worker-node> | grep Running

# CPU usage
top -bn1 | grep "Cpu(s)"

# Jellyfin sessions
curl -s http://192.168.4.63:30096/Sessions
```

### Sleep Decision

Node sleeps when:
- No active pods requiring node
- CPU usage < 10%
- No Jellyfin streams
- Idle time > 2 hours

## Sleep/Wake Cycle

### Sleep Sequence

1. Cordon worker nodes
2. Drain pods from workers
3. Scale down non-essential deployments
4. Send suspend command to workers

```bash
# Example sleep command
kubectl cordon storagenodet3500
kubectl drain storagenodet3500 --ignore-daemonsets --delete-emptydir-data
ssh root@192.168.4.61 "systemctl suspend"
```

### Wake Sequence

1. Send WoL magic packets
2. Wait for SSH availability
3. Uncordon nodes
4. Pods automatically reschedule

```bash
# Example wake script
wakeonlan b8:ac:6f:7e:6c:9d
while ! ssh -o ConnectTimeout=5 root@192.168.4.61 "echo ok" 2>/dev/null; do
  sleep 5
done
kubectl uncordon storagenodet3500
```

## Testing Sleep/Wake

### Run Test

```bash
./tests/test-sleep-wake-cycle.sh
```

**Warning**: This test actually sleeps nodes.

### Manual Test

```bash
# 1. Record state
kubectl get nodes
kubectl get pods -o wide -A

# 2. Sleep node
ssh root@192.168.4.61 "systemctl suspend"

# 3. Wait 30 seconds

# 4. Wake node
wakeonlan b8:ac:6f:7e:6c:9d

# 5. Verify recovery
kubectl get nodes
kubectl get pods -o wide -A
```

## BIOS Configuration

### Enable WoL in BIOS

1. Enter BIOS setup
2. Find Power Management settings
3. Enable "Wake on LAN" or "Power on by PCI-E"
4. Save and exit

### Common BIOS Names

- Wake on LAN
- Wake on PCI-E
- Power on by network
- Resume on LAN

## Network Requirements

### Switch Configuration

- Support for broadcast packets
- Same VLAN/subnet for all nodes
- No aggressive port security blocking magic packets

### Firewall

WoL uses UDP port 9 (or 7):
- Usually works without firewall rules
- Magic packets are Layer 2 broadcast

## Troubleshooting

### Node Won't Wake

1. **Check BIOS**: WoL enabled?
2. **Check network**: Node connected?
3. **Check MAC**: Correct MAC address?
4. **Test locally**: 
   ```bash
   ssh root@<node> "ethtool -s eth0 wol g"
   ```

### Node Won't Sleep

1. **Check workloads**: Active pods?
2. **Check permissions**: Can run systemctl suspend?
3. **Check power settings**:
   ```bash
   ssh root@<node> "cat /sys/power/state"
   ```

### View Sleep Logs

```bash
# On sleeping node (after wake)
journalctl -b -1 | grep -i suspend
```

### Collect Wake Logs

```bash
./scripts/vmstation-collect-wake-logs.sh
```

## Disable Auto-Sleep

### Temporarily

```bash
sudo systemctl stop vmstation-autosleep.timer
```

### Permanently

```bash
sudo systemctl disable vmstation-autosleep.timer
```

### For Specific Node

Add to node configuration or inventory:
```yaml
auto_sleep_enabled: false
```

## Energy Savings

### Estimated Savings

| Node | Power (Active) | Power (Sleep) | Savings |
|------|---------------|---------------|---------|
| storagenodet3500 | ~100W | ~5W | 95W |
| homelab | ~150W | ~10W | 140W |

### Annual Estimate

With 16 hours daily sleep:
- ~$150-300/year savings (varies by electricity cost)

## Related Documentation

- [Day-2 Operations](day-2-operations.md)
- [Cluster Management](cluster-management.md)
- [Troubleshooting](../troubleshooting/common-issues.md)
