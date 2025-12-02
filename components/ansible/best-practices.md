# Ansible Best Practices

Guidelines for working with Ansible in VMStation.

## Inventory Management

### Use YAML Format

```yaml
# /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml
all:
  vars:
    kubernetes_version: "1.29"
  children:
    monitoring_nodes:
      hosts:
        masternode:
          ansible_host: 192.168.4.63
```

### Absolute Paths

Always use absolute paths for SSH keys:

```yaml
ansible_ssh_private_key_file: /root/.ssh/id_k3s
# Not: ~/.ssh/id_k3s
```

### Group Variables

Use group_vars for shared settings:

```yaml
# ansible/inventory/group_vars/all.yml
kubernetes_version: "1.29"
pod_network_cidr: "10.244.0.0/16"
```

## Playbook Best Practices

### Idempotency

Design tasks to be safe when run multiple times:

```yaml
# Good - idempotent
- name: Ensure directory exists
  ansible.builtin.file:
    path: /etc/myapp
    state: directory
    mode: '0755'

# Bad - not idempotent
- name: Create directory
  ansible.builtin.command: mkdir /etc/myapp
```

### Check Mode Support

Use `check_mode` compatible modules:

```yaml
- name: Copy config
  ansible.builtin.copy:
    src: config.yml
    dest: /etc/myapp/config.yml
  # Supports --check flag
```

### Changed When

Set appropriate changed conditions:

```yaml
- name: Check cluster status
  ansible.builtin.command: kubectl get nodes
  register: cluster_status
  changed_when: false  # Read-only command
```

### Failed When

Define failure conditions:

```yaml
- name: Validate deployment
  ansible.builtin.uri:
    url: http://localhost:8080/health
  register: health
  failed_when: health.status != 200
```

## Task Organization

### Use Blocks

Group related tasks:

```yaml
- name: Setup Kubernetes
  block:
    - name: Install kubeadm
      ansible.builtin.apt:
        name: kubeadm
        state: present
    
    - name: Initialize cluster
      ansible.builtin.command: kubeadm init
  rescue:
    - name: Reset on failure
      ansible.builtin.command: kubeadm reset -f
```

### Tags

Use tags for selective execution:

```yaml
- name: Deploy monitoring
  tags: [monitoring]
  block:
    - name: Deploy Prometheus
      # ...
```

Run with tags:
```bash
ansible-playbook site.yml --tags monitoring
```

### Handlers

Use handlers for service restarts:

```yaml
tasks:
  - name: Update config
    ansible.builtin.copy:
      src: config.yml
      dest: /etc/myapp/config.yml
    notify: Restart myapp

handlers:
  - name: Restart myapp
    ansible.builtin.service:
      name: myapp
      state: restarted
```

## Variables

### Variable Precedence

(Highest to lowest)
1. Extra vars (`-e`)
2. Task vars
3. Block vars
4. Role vars
5. Play vars
6. Host vars
7. Group vars
8. Role defaults

### Defaults in Roles

```yaml
# roles/myrole/defaults/main.yml
myrole_port: 8080
myrole_enabled: true
```

### Sensitive Data

Use Ansible Vault:

```bash
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
```

Reference:
```yaml
vars_files:
  - secrets.yml
```

## Error Handling

### Ignore Errors

```yaml
- name: Check optional service
  ansible.builtin.command: systemctl status optional
  ignore_errors: true
```

### Retry Logic

```yaml
- name: Wait for API
  ansible.builtin.uri:
    url: http://localhost:6443/healthz
  register: result
  until: result.status == 200
  retries: 30
  delay: 10
```

## Testing

### Check Mode

```bash
ansible-playbook site.yml --check
```

### Diff Mode

```bash
ansible-playbook site.yml --diff
```

### Syntax Check

```bash
ansible-playbook site.yml --syntax-check
```

### Lint

```bash
ansible-lint playbooks/
```

## Performance

### Parallel Execution

```yaml
# ansible.cfg
[defaults]
forks = 10
```

### Pipelining

```yaml
# ansible.cfg
[ssh_connection]
pipelining = True
```

### Fact Caching

```yaml
# ansible.cfg
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/facts_cache
fact_caching_timeout = 86400
```

## Related Documentation

- [Playbooks](playbooks.md)
- [Roles](roles.md)
- [Deployment](../../deployment/README.md)
