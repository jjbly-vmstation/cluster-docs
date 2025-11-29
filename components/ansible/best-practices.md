# Ansible Best Practices

This guide covers Ansible best practices used in VMStation.

## Project Structure

```
ansible/
├── ansible.cfg           # Ansible configuration
├── inventory/
│   └── hosts.yml         # Inventory file
├── playbooks/            # Playbooks
├── roles/                # Roles
├── files/                # Static files
├── templates/            # Jinja2 templates
└── artifacts/            # Deployment logs
```

## Inventory Best Practices

### Use Absolute Paths

```yaml
# Correct
ansible_ssh_private_key_file: /root/.ssh/id_k3s

# Wrong - can cause issues
ansible_ssh_private_key_file: ~/.ssh/id_k3s
```

### Group Variables

```yaml
all:
  vars:
    kubernetes_version: "1.29"
    
monitoring_nodes:
  hosts:
    masternode:
      ansible_host: 192.168.4.63
```

### Host-Specific Variables

```yaml
storage_nodes:
  hosts:
    storagenodet3500:
      mac_address: "b8:ac:6f:7e:6c:9d"
      auto_sleep: true
```

## Playbook Best Practices

### Idempotency

Make playbooks safe to run multiple times:

```yaml
- name: Create directory
  file:
    path: /srv/data
    state: directory
    mode: '0755'
  # Creates if missing, no change if exists
```

### Error Handling

```yaml
- name: Check if kubeadm initialized
  stat:
    path: /etc/kubernetes/admin.conf
  register: kubeadm_init

- name: Initialize kubeadm
  command: kubeadm init
  when: not kubeadm_init.stat.exists
```

### Use Blocks for Error Handling

```yaml
- name: Deploy application
  block:
    - name: Apply manifest
      kubernetes.core.k8s:
        src: app.yaml
        
    - name: Wait for ready
      kubernetes.core.k8s_info:
        kind: Pod
        label_selectors:
          - app=myapp
  rescue:
    - name: Collect diagnostics
      shell: kubectl describe pod -l app=myapp
      register: diagnostics
      
    - name: Save diagnostics
      copy:
        content: "{{ diagnostics.stdout }}"
        dest: /tmp/diagnostics.log
```

### Register and Debug

```yaml
- name: Get cluster info
  command: kubectl get nodes
  register: nodes_output

- name: Display nodes
  debug:
    var: nodes_output.stdout_lines
```

## Role Best Practices

### Use Defaults

```yaml
# roles/my-role/defaults/main.yml
my_setting: default_value
my_port: 8080
```

### Use Handlers

```yaml
# roles/my-role/tasks/main.yml
- name: Update config
  template:
    src: config.j2
    dest: /etc/myapp/config.yaml
  notify: restart myapp

# roles/my-role/handlers/main.yml
- name: restart myapp
  systemd:
    name: myapp
    state: restarted
```

### Document Roles

Include README.md in each role:

```markdown
# My Role

## Description
What this role does.

## Requirements
What's needed before running.

## Variables
Available variables and defaults.

## Usage
Example playbook usage.
```

## Security Best Practices

### Use Ansible Vault

```bash
# Create vault
ansible-vault create secrets.yml

# Edit vault
ansible-vault edit secrets.yml

# Use in playbook
ansible-playbook playbook.yml --ask-vault-pass
```

### Don't Log Sensitive Data

```yaml
- name: Set password
  command: "set-password {{ vault_password }}"
  no_log: true
```

### Use become Appropriately

```yaml
- name: Install package
  apt:
    name: package
  become: true

- name: User-level task
  command: user-command
  become: false
```

## Performance Best Practices

### Use Async for Long Tasks

```yaml
- name: Long running task
  command: /long/running/script.sh
  async: 300
  poll: 10
```

### Optimize Fact Gathering

```yaml
- hosts: all
  gather_facts: false  # If facts not needed
  
# Or gather specific facts
- hosts: all
  gather_facts: true
  gather_subset:
    - network
```

### Use Pipelining

In ansible.cfg:
```ini
[ssh_connection]
pipelining = True
```

## Testing

### Dry Run

```bash
ansible-playbook playbook.yml --check
```

### Syntax Check

```bash
ansible-playbook playbook.yml --syntax-check
```

### Verbose Mode

```bash
ansible-playbook playbook.yml -vvv
```

### Limit Hosts

```bash
ansible-playbook playbook.yml --limit homelab
```

## Logging

### Configure Logging

In ansible.cfg:
```ini
[defaults]
log_path = ansible/artifacts/ansible.log
```

### Task Logging

```yaml
- name: Run script
  command: /path/to/script.sh
  register: result

- name: Save output
  copy:
    content: "{{ result.stdout }}"
    dest: "{{ artifact_dir }}/script.log"
```

## Related Documentation

- [Playbooks](playbooks.md)
- [Roles](roles.md)
- [Cluster Deployment](../../deployment/cluster-deployment.md)
