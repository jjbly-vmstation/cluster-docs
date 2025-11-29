# Power Management

Auto-sleep and Wake-on-LAN configuration.

## Overview

VMStation uses power management to save energy:
- **Auto-sleep** - Worker nodes sleep after idle period
- **Wake-on-LAN** - Remote wake via magic packets
- **masternode** - Always-on (control plane)

## Configuration

### Setup Auto-Sleep

```bash
./deploy.sh setup
```

This configures:
- Sleep monitoring scripts
- Cron job for idle detection
- Wake-on-LAN handlers
- systemd timers

## Auto-Sleep Behavior

### Idle Detection

Monitored every hour:
1. Check Jellyfin activity
2. Check CPU usage
3. Check active sessions
4. If idle for 2+ hours → trigger sleep

### Sleep Sequence

1. Cordon worker nodes
2. Drain pods
3. Scale down non-essential deployments
4. Suspend nodes (`systemctl suspend`)

### Wake Sequence

1. Timer triggers or manual wake
2. Send WoL magic packets
3. Monitor SSH availability
4. Uncordon nodes
5. Pods restart automatically

## Wake-on-LAN

### MAC Addresses

| Node | MAC Address |
|------|-------------|
| storagenodet3500 | b8:ac:6f:7e:6c:9d |
| homelab | d0:94:66:30:d6:63 |

### Manual Wake

```bash
# Install wakeonlan
apt install wakeonlan

# Wake nodes
wakeonlan b8:ac:6f:7e:6c:9d  # storagenodet3500
wakeonlan d0:94:66:30:d6:63  # homelab
```

### Wake Script

```bash
#!/bin/bash
# /root/scripts/wake-nodes.sh

echo "Waking storagenodet3500..."
wakeonlan -i 192.168.4.255 b8:ac:6f:7e:6c:9d

echo "Waking homelab..."
wakeonlan -i 192.168.4.255 d0:94:66:30:d6:63

# Wait for nodes
sleep 60

# Verify
for node in 192.168.4.61 192.168.4.62; do
    if ping -c 1 $node > /dev/null 2>&1; then
        echo "$node is up"
    else
        echo "$node is still down"
    fi
done
```

## BIOS Configuration

### Enable WoL

1. Enter BIOS setup (F2/Del during boot)
2. Navigate to Power Management
3. Enable "Wake on LAN" or "Wake on PCI"
4. Enable "Deep Sleep" disabled (if applicable)
5. Save and exit

### Network Interface

Verify WoL is enabled:

```bash
ethtool eth0 | grep Wake-on
# Should show: Wake-on: g
```

Enable WoL:

```bash
ethtool -s eth0 wol g
```

Make persistent (Debian):

```bash
# /etc/network/interfaces
auto eth0
iface eth0 inet dhcp
    post-up ethtool -s eth0 wol g
```

## Sleep Management Scripts

### Check Sleep Status

```bash
# View sleep state on node
ssh root@192.168.4.61 "systemctl status sleep.target"
```

### Manual Sleep

```bash
# On the node
systemctl suspend
```

### Prevent Sleep

```bash
# Temporarily prevent sleep
systemd-inhibit --what=sleep --who=admin --why="Maintenance" sleep infinity &
```

## Testing

### Test Sleep/Wake Cycle

```bash
./tests/test-sleep-wake-cycle.sh
```

This will:
1. Record cluster state
2. Trigger sleep
3. Send WoL packets
4. Measure wake time
5. Validate services

### Test WoL Only

```bash
# Put node to sleep
ssh root@192.168.4.61 "systemctl suspend"

# Wait a moment
sleep 5

# Wake node
wakeonlan b8:ac:6f:7e:6c:9d

# Verify
ping 192.168.4.61
```

## Troubleshooting

### Node Not Waking

1. **Check BIOS settings** - WoL enabled?
2. **Check NIC settings** - `ethtool eth0 | grep Wake-on`
3. **Check MAC address** - `ip link show eth0`
4. **Try from same subnet** - WoL requires L2 connectivity
5. **Check switch** - Some switches block magic packets

### Node Sleeping Unexpectedly

```bash
# Check sleep timers
ssh root@192.168.4.61 "systemctl list-timers"

# Check sleep scripts
ssh root@192.168.4.61 "cat /etc/cron.d/auto-sleep"

# Reset counters
ssh root@192.168.4.61 "echo 0 > /var/lib/vmstation/idle-counter"
```

### Pods Not Restarting After Wake

```bash
# Check node status
kubectl get nodes

# Uncordon if needed
kubectl uncordon storagenodet3500
kubectl uncordon homelab

# Check pods
kubectl get pods -A | grep -v Running
```

## Monitoring

### Prometheus Metrics

```promql
# Node up time
node_time_seconds - node_boot_time_seconds

# Check if node is up
up{job="node-exporter"}
```

### Grafana Dashboard

Create panel showing node availability over time.

## Related Documentation

- [Day-2 Operations](day-2-operations.md)
- [Cluster Management](cluster-management.md)
- [Prerequisites](../getting-started/prerequisites.md)
