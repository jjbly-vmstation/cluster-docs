# TODO

Current tasks and backlog for VMStation.

## Current Workflow

```bash
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Immediate Tasks

### Monitoring & Observability

- [ ] Deploy and configure IPMI Exporter on homelab node
  - Follow guide: `docs/RHEL10_HOMELAB_METRICS_SETUP.md`
  - Install IPMI exporter on 192.168.4.62
  - Configure IPMI credentials

- [ ] Verify Grafana datasource connectivity
  - Test Prometheus datasource
  - Test Loki datasource

### Testing & Validation

- [ ] Add WoL testing to ansible playbook
  - Sleep nodes
  - Send WoL magic packet
  - Measure wake time

- [ ] Reduce curl output verbosity
  - Use `curl -s` with custom formatting
  - Format: "curl ip:port ok" or "curl ip:port ERROR"

## Infrastructure Tasks

### SSO / Kerberos (FreeIPA)

Goal: Run Kerberos/FreeIPA identity service for home network SSO.

- [ ] Decide deployment target (Kubernetes pod vs dedicated VM)
- [ ] Create manifest: `manifests/idm/freeipa-statefulset.yaml`
- [ ] Provision storage on masternode
- [ ] Deploy FreeRADIUS for Wi-Fi authentication
- [ ] Configure Samba/SSSD on storagenodet3500
- [ ] Backup & DR automation

## New Infrastructure

### Malware Analysis Lab

Create Terraform infrastructure for isolated analysis environment:

- [ ] Create initial Terraform scaffold
- [ ] Document resource requirements
- [ ] Create network isolation design
- [ ] Deploy Windows Server VMs
- [ ] Deploy Enterprise Linux VMs
- [ ] Configure network infrastructure
- [ ] Deploy IDS/IPS
- [ ] Deploy RKE2 + Splunk Enterprise

## Fixed Issues (Recent)

### ✅ January 2025

- [x] Kubespray deployment path added
- [x] Documentation consolidation
- [x] preflight-rhel10 role created

### ✅ October 2025

- [x] CNI Plugin Installation fixed
- [x] Blackbox Exporter CrashLoopBackOff resolved
- [x] Loki CrashLoopBackOff fixed
- [x] Jellyfin Pod scheduling fixed
- [x] WoL Validation SSH Error resolved

### ✅ Dashboard Improvements

- [x] Enhanced Loki Logs & Aggregation dashboard
- [x] Created Syslog Infrastructure Monitoring dashboard
- [x] Created CoreDNS Performance & Health dashboard

## Notes

- IPMI monitoring requires credentials
- Malware lab must remain isolated from production
- WoL testing will validate power management

## Related Documentation

- [Completed Features](completed-features.md)
- [Future Plans](future-plans.md)
