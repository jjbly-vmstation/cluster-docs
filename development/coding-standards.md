# Coding Standards

Code style guidelines for VMStation.

## General Principles

- **Clarity** over cleverness
- **Consistency** with existing code
- **Documentation** for complex logic
- **Testability** in design

## Ansible

### Task Names

Descriptive, action-oriented:

```yaml
# Good
- name: Install Kubernetes binaries
- name: Ensure directory exists
- name: Configure chrony time sync

# Bad
- name: Task 1
- name: Do stuff
```

### YAML Formatting

```yaml
# 2-space indentation
tasks:
  - name: Example task
    ansible.builtin.file:
      path: /etc/myapp
      state: directory
      mode: '0755'
```

### Module Usage

Use FQCN (Fully Qualified Collection Names):

```yaml
# Good
- name: Copy file
  ansible.builtin.copy:
    src: config.yml
    dest: /etc/app/config.yml

# Avoid
- name: Copy file
  copy:
    src: config.yml
    dest: /etc/app/config.yml
```

### Idempotency

Design tasks to be safe when run multiple times:

```yaml
# Good - idempotent
- name: Ensure package installed
  ansible.builtin.apt:
    name: nginx
    state: present

# Bad - not idempotent
- name: Install package
  ansible.builtin.command: apt install nginx
```

### Variables

Descriptive names with prefixes:

```yaml
# Role variables
preflight_chrony_servers:
  - pool.ntp.org

# Play variables
kubernetes_version: "1.29"
```

## Shell Scripts

### Shebang and Options

```bash
#!/bin/bash
set -e  # Exit on error
set -u  # Treat unset variables as error
set -o pipefail  # Pipe failures
```

### Functions

```bash
# Function documentation
# Arguments:
#   $1 - Description
# Returns:
#   0 on success, 1 on failure
function validate_input() {
    local input="$1"
    if [[ -z "$input" ]]; then
        echo "Error: Input required" >&2
        return 1
    fi
    return 0
}
```

### Error Handling

```bash
if ! command -v kubectl &> /dev/null; then
    echo "Error: kubectl not found" >&2
    exit 1
fi
```

### Output

```bash
# Informational
echo "Installing packages..."

# Success
echo "✓ Installation complete"

# Error (to stderr)
echo "✗ Installation failed" >&2

# Warning
echo "⚠ Warning: Using default config"
```

### Variables

```bash
# Use descriptive names
KUBERNETES_VERSION="1.29"
POD_NETWORK_CIDR="10.244.0.0/16"

# Quote variables
echo "Using version: ${KUBERNETES_VERSION}"

# Local variables in functions
local result
```

## YAML Files

### Formatting

```yaml
# Top-level key at column 0
apiVersion: v1
kind: Service
metadata:
  # 2-space indentation
  name: prometheus
  namespace: monitoring
spec:
  # Consistent formatting
  ports:
    - name: web
      port: 9090
      targetPort: 9090
```

### Comments

```yaml
# Section comment
apiVersion: v1

metadata:
  name: prometheus
  # Inline comment for specific field
  namespace: monitoring  # Required for service discovery
```

## Documentation

### Markdown

```markdown
# Heading 1

## Heading 2

### Heading 3

**Bold** for emphasis

`code` for commands

```bash
# Code blocks with language
kubectl get pods
```

### Links

```markdown
[Link text](path/to/file.md)
[External](https://example.com)
```

### Tables

```markdown
| Column 1 | Column 2 |
|----------|----------|
| Value 1  | Value 2  |
```

## Git Commits

### Format

```
type: short description (max 50 chars)

Longer description if needed. Wrap at 72 characters.

- Bullet points for multiple changes
- Another change

Fixes #123
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `refactor`: Code refactoring
- `test`: Test changes
- `chore`: Maintenance

## File Organization

### Directory Structure

```
ansible/
├── inventory/
│   └── /srv/vmstation-org/cluster-setup/ansible/inventory/hosts.yml
├── playbooks/
│   └── deploy-cluster.yaml
└── roles/
    └── myrole/
        ├── defaults/main.yml
        ├── tasks/main.yml
        └── templates/
```

### Naming

- Use lowercase
- Use hyphens for multi-word names
- Descriptive names

```
# Good
deploy-monitoring-stack.yaml
validate-cluster-health.sh

# Bad
deploy.yaml
script1.sh
```

## Related Documentation

- [Contributing](contributing.md)
- [Development Workflow](development-workflow.md)
- [Testing Guide](testing-guide.md)
