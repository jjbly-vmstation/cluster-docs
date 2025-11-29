# Coding Standards

Code style guidelines for VMStation.

## General Principles

- Readability over cleverness
- Consistent formatting
- Self-documenting code
- Comments where necessary

## Shell Scripts

### Shebang

```bash
#!/bin/bash
```

### Error Handling

```bash
set -e          # Exit on error
set -u          # Error on undefined variables
set -o pipefail # Pipe failures propagate
```

### Variables

```bash
# Use descriptive names
NODE_NAME="masternode"
TIMEOUT_SECONDS=60

# Quote variables
echo "${NODE_NAME}"

# Local variables in functions
my_function() {
    local result=""
}
```

### Functions

```bash
# Clear naming
check_node_status() {
    local node=$1
    kubectl get node "$node" -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
}
```

### Comments

```bash
# Brief description of what follows
if condition; then
    # Explain why, not what
    command
fi
```

## Ansible

### Playbooks

```yaml
- name: Descriptive task name
  module:
    parameter: value
  register: result
  when: condition
```

### Variables

```yaml
# In defaults/main.yml
my_setting: default_value

# In tasks
setting: "{{ my_setting }}"
```

### Handlers

```yaml
handlers:
  - name: restart service
    systemd:
      name: myservice
      state: restarted
```

### Error Handling

```yaml
- name: Task with error handling
  block:
    - name: Try this
      command: might-fail
  rescue:
    - name: Handle failure
      debug:
        msg: "Failed, recovering"
```

## YAML

### Formatting

```yaml
# Use 2 spaces for indentation
key:
  nested_key: value
  
# Lists
items:
  - item1
  - item2

# Long strings
description: |
  This is a long
  multi-line string
```

### Kubernetes Manifests

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: default
  labels:
    app: my-app
spec:
  replicas: 1
```

## Markdown

### Headings

```markdown
# Main Title

## Section

### Subsection
```

### Code Blocks

````markdown
```bash
command here
```
````

### Lists

```markdown
- Unordered item
  - Nested item

1. Ordered item
2. Second item
```

### Tables

```markdown
| Column 1 | Column 2 |
|----------|----------|
| Data 1   | Data 2   |
```

## Git

### Commit Messages

```
type: brief description (50 chars)

Optional detailed description (72 chars/line)

- Bullet points for specifics
- References: #issue
```

### Branch Names

```
feature/add-monitoring
fix/prometheus-permissions
docs/update-readme
```

## Related Documentation

- [Contributing](contributing.md)
- [Development Workflow](development-workflow.md)
