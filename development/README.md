# Development

Development documentation for VMStation.

## Documents

- [Contributing](contributing.md) - How to contribute
- [Development Workflow](development-workflow.md) - Development process
- [Testing Guide](testing-guide.md) - Testing procedures
- [AI Agent Implementation](ai-agent-implementation.md) - AI agent report
- [Coding Standards](coding-standards.md) - Code style guidelines

## Quick Reference

### Repository Structure

```
VMStation/
├── deploy.sh                    # Main deployment script
├── ansible/
│   ├── inventory/hosts.yml      # Cluster inventory
│   ├── playbooks/               # Ansible playbooks
│   └── roles/                   # Ansible roles
├── scripts/
│   ├── validate-monitoring-stack.sh
│   ├── diagnose-monitoring-stack.sh
│   └── run-kubespray.sh
├── tests/
│   ├── test-complete-validation.sh
│   └── test-sleep-wake-cycle.sh
├── manifests/
│   └── monitoring/              # Kubernetes manifests
└── docs/                        # Documentation
```

### Development Environment

- Ansible 2.9+
- Python 3.8+
- kubectl
- Access to test cluster

## Related Documentation

- [Getting Started](../getting-started/README.md)
- [Architecture](../architecture/README.md)
- [Reference](../reference/README.md)
