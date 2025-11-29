# Testing Guide

Testing procedures for VMStation.

## Test Categories

### Syntax Tests

Validate code syntax without execution.

```bash
# Ansible playbooks
ansible-playbook --syntax-check ansible/playbooks/*.yaml

# Shell scripts
bash -n scripts/*.sh

# YAML files
yamllint ansible/
```

### Dry Run Tests

Test execution without making changes.

```bash
./deploy.sh debian --check
./deploy.sh monitoring --check
./deploy.sh infrastructure --check
```

### Validation Tests

Verify deployed components work correctly.

```bash
# Complete validation
./tests/test-complete-validation.sh

# Monitoring validation
./scripts/validate-monitoring-stack.sh

# Time sync validation
./tests/validate-time-sync.sh
```

### Integration Tests

Test component interactions.

```bash
# Sleep/wake cycle
./tests/test-sleep-wake-cycle.sh

# Monitoring access
./tests/test-monitoring-access.sh
```

## Running Tests

### Complete Suite

```bash
./tests/test-complete-validation.sh
```

### Individual Tests

```bash
# Monitoring health
./tests/test-monitoring-exporters-health.sh

# Loki validation
./tests/test-loki-validation.sh

# Auto-sleep configuration
./tests/test-autosleep-wake-validation.sh
```

### Idempotency Test

Verify deployments can run multiple times safely.

```bash
./tests/test-idempotence.sh 3
```

## Test Scripts

### test-complete-validation.sh

Comprehensive validation covering:
1. Node status
2. Pod health
3. Service availability
4. Monitoring endpoints

### validate-monitoring-stack.sh

Monitoring-specific checks:
1. Pod status
2. Endpoints populated
3. Health endpoints
4. Datasource connectivity

### test-sleep-wake-cycle.sh

**Warning**: Actually sleeps nodes.

1. Record state
2. Sleep nodes
3. Wake with WoL
4. Verify recovery

## Writing Tests

### Shell Script Template

```bash
#!/bin/bash
set -e

echo "=== Test: My Test ==="

# Setup
# ...

# Test
if condition; then
    echo "PASS: Description"
else
    echo "FAIL: Description"
    exit 1
fi

# Cleanup
# ...

echo "=== Test Complete ==="
```

### Assertions

```bash
# Check command success
if kubectl get nodes >/dev/null 2>&1; then
    echo "PASS: Cluster accessible"
else
    echo "FAIL: Cluster not accessible"
    exit 1
fi

# Check output
if kubectl get pods -A | grep -q Running; then
    echo "PASS: Pods running"
fi
```

## CI/CD

### Pre-commit

- Syntax check
- Linting
- Dry run

### Post-merge

- Full deployment
- Validation suite
- Integration tests

## Related Documentation

- [Development Workflow](development-workflow.md)
- [Contributing](contributing.md)
- [Diagnostic Procedures](../troubleshooting/diagnostic-procedures.md)
