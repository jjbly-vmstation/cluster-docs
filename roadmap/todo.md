# TODO

Current tasks and work items for VMStation.

## Current Workflow

```bash
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Immediate Tasks

### Monitoring

- [ ] **Deploy IPMI Exporter on homelab**
  - Follow: `docs/RHEL10_HOMELAB_METRICS_SETUP.md`
  - Port: 9290
  - Requires IPMI credentials

- [ ] **Verify Prometheus targets**
  - `192.168.4.62:9290` - ipmi-exporter
  - `192.168.4.62:9100` - node-exporter
  - `192.168.4.61:9100` - node-exporter
  - `192.168.4.63:9100` - node-exporter

### Testing

- [ ] Add WoL testing to ansible playbook
- [ ] Reduce curl output verbosity in scripts

## Infrastructure

### SSO / Kerberos

Goal: FreeIPA for SSO across home network.

- [ ] Decide deployment target (K8s pod vs dedicated VM)
- [ ] Create FreeIPA manifest
- [ ] Provision storage
- [ ] Configure time sync
- [ ] Network hardening
- [ ] Deploy FreeRADIUS for Wi-Fi auth
- [ ] Create service principals
- [ ] Configure Samba integration
- [ ] Setup backups
- [ ] Add monitoring

### Malware Analysis Lab

Terraform infrastructure for isolated malware analysis.

Components:
- [ ] Windows Server VMs (2019, 2022)
- [ ] Enterprise Linux VMs (Rocky, AlmaLinux)
- [ ] Network infrastructure (Cisco simulation)
- [ ] IDS/IPS (Suricata/Snort)
- [ ] RKE2 + Splunk Enterprise
- [ ] Enterprise security services

## Completed (Recent)

### January 2025

- [x] Kubespray integration
- [x] Documentation consolidation
- [x] Preflight RHEL10 role

### October 2025

- [x] CNI plugin installation fix
- [x] Blackbox exporter fix
- [x] Loki CrashLoopBackOff fix
- [x] Jellyfin scheduling fix
- [x] WoL SSH authentication fix
- [x] Enhanced Grafana dashboards
- [x] Syslog infrastructure dashboard
- [x] CoreDNS performance dashboard

## Future Plans

See [Future Plans](future-plans.md) for long-term roadmap.

## Related Documentation

- [Completed Features](completed-features.md)
- [Future Plans](future-plans.md)
- [Development](../development/README.md)
