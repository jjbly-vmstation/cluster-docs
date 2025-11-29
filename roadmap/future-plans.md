# Future Plans

Planned features and improvements for VMStation.

## Short-Term (Next Month)

### IPMI Monitoring

- Deploy IPMI exporter on homelab
- Configure hardware monitoring dashboard
- Add temperature/fan alerts

### Enhanced Alerting

- Configure Prometheus alerting rules
- Add alert notifications
- Create alert dashboard

### Documentation Versioning

- Consider MkDocs or Docusaurus
- Add version selector
- Automated deployment

## Medium-Term (3-6 Months)

### Malware Analysis Lab

Complete isolated environment for security research:

- Windows Server VMs
- Enterprise Linux VMs
- Network isolation
- Splunk Enterprise integration
- IDS/IPS deployment

### SSO / Kerberos

- FreeIPA deployment
- Samba integration
- Wi-Fi authentication via RADIUS

### Backup Automation

- Scheduled backups
- Off-site backup
- Restore testing

## Long-Term (6+ Months)

### High Availability

- Additional control plane nodes
- External etcd cluster
- Load balancer for API

### GitOps

- ArgoCD or Flux
- Declarative application deployment
- Git-based configuration

### Multi-Cluster

- Federation between clusters
- Centralized monitoring
- Cross-cluster networking

## Wishlist

### Infrastructure

- [ ] Distributed storage (Ceph, Longhorn)
- [ ] Service mesh (Istio, Linkerd)
- [ ] Secrets management (Vault)
- [ ] Certificate automation (cert-manager)

### Monitoring

- [ ] Thanos for HA Prometheus
- [ ] Tempo for distributed tracing
- [ ] Custom metrics exporters
- [ ] SLO dashboards

### Security

- [ ] Network policies
- [ ] Pod security policies
- [ ] Image scanning
- [ ] Runtime security

### Developer Experience

- [ ] Local development environment
- [ ] CI/CD pipelines
- [ ] Test environment provisioning
- [ ] Development documentation

## Not Planned

- Cloud integration (staying on-premises)
- Managed Kubernetes (prefer self-managed)

## Contributing Ideas

Open an issue to suggest new features!

## Related Documentation

- [TODO](todo.md)
- [Completed Features](completed-features.md)
