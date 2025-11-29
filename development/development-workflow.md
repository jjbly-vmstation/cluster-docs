# Development Workflow

Development process for VMStation.

## Branch Strategy

### Branches

| Branch | Purpose |
|--------|---------|
| `main` | Stable, production-ready |
| `feature/*` | New features |
| `fix/*` | Bug fixes |
| `auto/*` | Automated changes |

### Flow

1. Create branch from `main`
2. Develop and test
3. Submit PR
4. Review and merge

## Development Cycle

### 1. Plan

- Review requirements
- Check existing issues
- Design approach

### 2. Implement

```bash
# Create branch
git checkout -b feature/my-feature

# Make changes
# ...

# Commit
git add .
git commit -m "feat: description"
```

### 3. Test

```bash
# Syntax check
ansible-playbook --syntax-check playbook.yml

# Dry run
./deploy.sh <command> --check

# Full test
./tests/test-complete-validation.sh
```

### 4. Document

- Update relevant docs
- Add inline comments
- Update README if needed

### 5. Submit

```bash
git push origin feature/my-feature
# Create PR on GitHub
```

## Testing Workflow

### Local Testing

```bash
# Run specific test
./tests/test-<name>.sh

# Run all validation
./tests/test-complete-validation.sh

# Run monitoring validation
./scripts/validate-monitoring-stack.sh
```

### Idempotency Testing

```bash
# Test deployment is idempotent
./tests/test-idempotence.sh 3
```

### Pre-Deployment

```bash
# Check before deploying
./tests/pre-deployment-checklist.sh
```

## Deployment Testing

### Fresh Deployment

```bash
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray
./scripts/validate-monitoring-stack.sh
```

### Component Deployment

```bash
# Test specific component
./deploy.sh monitoring --check
./deploy.sh monitoring
./scripts/validate-monitoring-stack.sh
```

## Debugging

### Check Logs

```bash
# Ansible logs
cat ansible/artifacts/<playbook>.log

# Pod logs
kubectl logs <pod> -n <namespace>

# System logs
journalctl -xeu kubelet
```

### Diagnostics

```bash
# Run diagnostics
./scripts/diagnose-monitoring-stack.sh

# Review output
cat /tmp/monitoring-diagnostics-*/00-ANALYSIS-AND-RECOMMENDATIONS.txt
```

## Release Process

### Version Tagging

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

### Release Notes

Include:
- New features
- Bug fixes
- Breaking changes
- Upgrade instructions

## Tools

### Recommended

- VS Code with Ansible extension
- ShellCheck for bash scripts
- yamllint for YAML files
- kubectl with bash completion

### Linting

```bash
# YAML
yamllint ansible/

# Shell
shellcheck scripts/*.sh

# Ansible
ansible-lint playbooks/
```

## Related Documentation

- [Contributing](contributing.md)
- [Testing Guide](testing-guide.md)
- [Coding Standards](coding-standards.md)
