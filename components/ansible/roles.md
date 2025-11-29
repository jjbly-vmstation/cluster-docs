# Ansible Roles

Documentation for VMStation Ansible roles.

## Available Roles

### preflight-rhel10

Prepares RHEL10 nodes for Kubernetes deployment.

**Location:** `ansible/roles/preflight-rhel10/`

**Tasks:**
- Install Python3 and packages
- Configure chrony for time sync
- Setup sudoers for ansible user
- Open firewall ports
- Configure SELinux (permissive)
- Load kernel modules
- Apply sysctl settings
- Disable swap

**Variables:**

```yaml
# defaults/main.yml
preflight_chrony_servers:
  - pool.ntp.org
  - time.cloudflare.com

preflight_firewall_ports:
  - 6443/tcp   # Kubernetes API
  - 10250/tcp  # Kubelet
  - 10251/tcp  # Scheduler
  - 10252/tcp  # Controller
  - 30000-32767/tcp  # NodePort

preflight_selinux_mode: permissive

preflight_kernel_modules:
  - br_netfilter
  - overlay
  - ip_vs
  - ip_vs_rr
  - ip_vs_wrr
  - ip_vs_sh

preflight_sysctl_settings:
  net.bridge.bridge-nf-call-iptables: 1
  net.bridge.bridge-nf-call-ip6tables: 1
  net.ipv4.ip_forward: 1
```

**Usage:**

```yaml
- hosts: compute_nodes
  roles:
    - preflight-rhel10
```

Or:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/run-preflight-rhel10.yml
```

## Role Structure

```
ansible/roles/preflight-rhel10/
├── README.md
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
└── templates/
    └── chrony.conf.j2
```

### defaults/main.yml

Default variable values, can be overridden.

### handlers/main.yml

Handlers for service restarts:

```yaml
- name: Restart chronyd
  ansible.builtin.service:
    name: chronyd
    state: restarted
```

### tasks/main.yml

Main task file with all role tasks.

### templates/

Jinja2 templates for configuration files.

## Creating Custom Roles

### Scaffold Role

```bash
cd ansible/roles
ansible-galaxy init my-new-role
```

### Role Best Practices

1. **Idempotent** - Safe to run multiple times
2. **Documented** - README with usage
3. **Tested** - Validate with molecule
4. **Parameterized** - Use variables for flexibility

## Related Documentation

- [Playbooks](playbooks.md)
- [Best Practices](best-practices.md)
