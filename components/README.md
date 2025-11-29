# Components

This section documents the individual components that make up VMStation.

## Categories

### [Ansible](ansible/)
Automation and infrastructure as code components.
- [Playbooks](ansible/playbooks.md)
- [Roles](ansible/roles.md)
- [Best Practices](ansible/best-practices.md)

### [Monitoring](monitoring/)
Observability and monitoring stack components.
- [Prometheus](monitoring/prometheus.md)
- [Grafana](monitoring/grafana.md)
- [Loki](monitoring/loki.md)
- [Dashboards](monitoring/dashboards.md)
- [Exporters](monitoring/exporters.md)

### [Infrastructure](infrastructure/)
Core infrastructure services.
- [NTP/Chrony](infrastructure/ntp-chrony.md)
- [Syslog](infrastructure/syslog.md)
- [Kerberos](infrastructure/kerberos.md)
- [DNS](infrastructure/dns.md)

### [Applications](applications/)
Application workloads.
- [Jellyfin](applications/jellyfin.md)

### [Storage](storage/)
Storage components and configuration.
- [Storage Setup](storage/storage-setup.md)

## Component Overview

| Category | Component | Purpose |
|----------|-----------|---------|
| Monitoring | Prometheus | Metrics collection |
| Monitoring | Grafana | Visualization |
| Monitoring | Loki | Log aggregation |
| Monitoring | Promtail | Log shipping |
| Monitoring | Node Exporter | System metrics |
| Infrastructure | Chrony | Time sync |
| Infrastructure | CoreDNS | DNS resolution |
| Applications | Jellyfin | Media streaming |
| Storage | local-path | PV provisioning |
