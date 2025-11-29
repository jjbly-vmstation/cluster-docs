# Ansible Roles

This document covers the Ansible roles used in VMStation.

## Role Overview

| Role | Purpose | Target |
|------|---------|--------|
| preflight-rhel10 | Prepare RHEL10 for Kubernetes | homelab |

## preflight-rhel10

Prepares RHEL10 nodes for Kubernetes deployment.

### Purpose

The preflight role ensures RHEL10 nodes meet all requirements for Kubernetes installation via Kubespray or kubeadm.

### Tasks

1. **Install Python3 packages**
   - python3
   - python3-pip
   - Required for Ansible operations

2. **Configure Chrony**
   - Time synchronization
   - Critical for Kubernetes certificates

3. **Setup Sudoers**
   - NOPASSWD for ansible user
   - Required for automation

4. **Open Firewall Ports**
   - 6443 (API Server)
   - 10250 (Kubelet)
   - 30000-32767 (NodePort)
   - 2379-2380 (etcd)

5. **Configure SELinux**
   - Set to permissive
   - Can be configured to enforcing

6. **Load Kernel Modules**
   - br_netfilter
   - overlay
   - ip_vs modules

7. **Apply Sysctl Settings**
   - net.bridge.bridge-nf-call-iptables
   - net.bridge.bridge-nf-call-ip6tables
   - net.ipv4.ip_forward

8. **Disable Swap**
   - Required for Kubernetes
   - Removes swap entries from fstab

### Usage

```bash
ansible-playbook -i ansible/inventory/hosts.yml \
  ansible/playbooks/run-preflight-rhel10.yml
```

### Variables

Default variables in `defaults/main.yml`:

```yaml
# Chrony configuration
chrony_ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org

# SELinux mode
selinux_mode: permissive

# Firewall ports
kubernetes_ports:
  - 6443/tcp
  - 10250/tcp
  - 30000-32767/tcp
```

### Handlers

```yaml
handlers:
  - name: restart chronyd
    systemd:
      name: chronyd
      state: restarted
```

### Templates

**chrony.conf.j2:**
```jinja2
{% for server in chrony_ntp_servers %}
server {{ server }} iburst
{% endfor %}

driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

### Idempotency

The role is fully idempotent:
- Safe to run multiple times
- Only changes what's needed
- Reports when changes are made

### Directory Structure

```
ansible/roles/preflight-rhel10/
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── chrony.conf.j2
└── README.md
```

## Creating New Roles

### Role Scaffold

```bash
ansible-galaxy role init ansible/roles/my-role
```

### Best Practices

1. **Use defaults for variables**
2. **Include handlers for service restarts**
3. **Make tasks idempotent**
4. **Document in README.md**
5. **Test with --check mode**

## Related Documentation

- [Ansible Playbooks](playbooks.md)
- [Best Practices](best-practices.md)
- [Kubespray Deployment](../../deployment/kubespray-deployment.md)
