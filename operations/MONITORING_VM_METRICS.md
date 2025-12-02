# Monitoring VM Metrics from Hypervisors

## Overview
To efficiently monitor all virtual machines (VMs) managed by your hypervisor (Proxmox, VMware, KVM/libvirt, etc.), use a hypervisor-level exporter that automatically discovers and exports metrics for all VMs. This approach eliminates the need to update Kubernetes playbooks or manifests every time a new VM is added.

## Best Practice
- **Deploy a hypervisor-level exporter** (e.g., Proxmox exporter, vSphere exporter, libvirt exporter) on the hypervisor or a management VM.
- **Configure Prometheus** to scrape the exporter endpoint. Prometheus will automatically collect metrics for all VMs, including new ones, as long as the exporter is running.
- **No need to update Kubernetes or Ansible** for each new VM. The exporter handles dynamic discovery.

## Example Exporters
- **Proxmox:** [prometheus-pve-exporter](https://github.com/prometheus-pve/prometheus-pve-exporter)
- **VMware vSphere:** [vsphere_exporter](https://github.com/pryorda/vmware_exporter)
- **KVM/libvirt:** [prometheus-libvirt-exporter](https://github.com/kumina/libvirt-prometheus-exporter)

## Prometheus Configuration Example
```yaml
scrape_configs:
  - job_name: 'proxmox'
    static_configs:
      - targets: ['<proxmox-host>:9221']
  - job_name: 'vsphere'
    static_configs:
      - targets: ['<vcenter-host>:9272']
  - job_name: 'libvirt'
    static_configs:
      - targets: ['<libvirt-host>:9177']
```

## Optional: Dynamic Inventory for Ansible
If you want to keep your Ansible inventory up to date, use dynamic inventory plugins/scripts that query the hypervisor API.

## Summary
- Use a hypervisor-level exporter for VM metrics.
- Prometheus scrapes the exporter endpoint—no per-VM config needed.
- This ensures all VMs (including new ones) are monitored automatically.

---
Add this to your monitoring stack planning and reference this document when implementing or updating VM metrics collection.
