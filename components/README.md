# Components

Documentation for VMStation components.

## Sections

### [Ansible](ansible/)
- [Playbooks](ansible/playbooks.md) - Ansible playbook documentation
- [Roles](ansible/roles.md) - Ansible role documentation
- [Best Practices](ansible/best-practices.md) - Ansible best practices

### [Monitoring](monitoring/)
- [Prometheus](monitoring/prometheus.md) - Metrics collection
- [Grafana](monitoring/grafana.md) - Visualization
- [Loki](monitoring/loki.md) - Log aggregation
- [Dashboards](monitoring/dashboards.md) - Dashboard configuration
- [Exporters](monitoring/exporters.md) - Metric exporters

### [Infrastructure](infrastructure/)
- [NTP/Chrony](infrastructure/ntp-chrony.md) - Time synchronization
- [Syslog](infrastructure/syslog.md) - Centralized logging
- [Kerberos](infrastructure/kerberos.md) - SSO and identity
- [DNS](infrastructure/dns.md) - CoreDNS configuration

### [Applications](applications/)
- [Jellyfin](applications/jellyfin.md) - Media server

### [Storage](storage/)
- [Storage Setup](storage/storage-setup.md) - Storage configuration

## Component Overview

| Component | Purpose | Port |
|-----------|---------|------|
| Prometheus | Metrics | 30090 |
| Grafana | Dashboards | 30300 |
| Loki | Logs | 31100 |
| Node Exporter | System metrics | 9100 |
| Jellyfin | Media streaming | 30096 |
| Chrony | Time sync | - |
| CoreDNS | DNS | 53 |

## Related Documentation

- [Architecture](../architecture/README.md)
- [Deployment](../deployment/README.md)
- [Troubleshooting](../troubleshooting/README.md)
