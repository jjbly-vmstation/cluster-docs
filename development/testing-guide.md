# Testing Guide

Testing procedures for VMStation.

## Test Categories

### Validation Tests

| Test | Purpose |
|------|---------|
| `test-complete-validation.sh` | Complete validation suite |
| `validate-monitoring-stack.sh` | Monitoring health |
| `pre-deployment-checklist.sh` | Pre-deployment checks |

### Component Tests

| Test | Purpose |
|------|---------|
| `test-autosleep-wake-validation.sh` | Auto-sleep config |
| `test-monitoring-exporters-health.sh` | Exporter health |
| `test-monitoring-access.sh` | Monitoring endpoints |
| `test-loki-validation.sh` | Loki health |

### Functional Tests

| Test | Purpose |
|------|---------|
| `test-sleep-wake-cycle.sh` | Power management |
| `test-idempotence.sh` | Deployment idempotency |
| `validate-time-sync.sh` | NTP synchronization |

## Running Tests

### Complete Validation

```bash
./tests/test-complete-validation.sh
```

Runs:
1. Configuration validation
2. Monitoring health
3. Sleep/wake cycle (optional)

### Monitoring Validation

```bash
./scripts/validate-monitoring-stack.sh
```

Checks:
1. Pod status
2. Service endpoints
3. PVC bindings
4. Health endpoints
5. DNS resolution
6. Container restarts
7. Log analysis

### Pre-Deployment

```bash
./tests/pre-deployment-checklist.sh
```

Validates:
- SSH connectivity
- Time synchronization
- Required binaries
- Network connectivity

## Specific Tests

### Auto-Sleep Validation

```bash
./tests/test-autosleep-wake-validation.sh
```

Checks:
- Sleep configuration
- WoL setup
- MAC addresses
- Timer configuration

### Monitoring Exporters

```bash
./tests/test-monitoring-exporters-health.sh
```

Checks:
- Node Exporter
- Blackbox Exporter
- Kube-state-metrics
- Prometheus targets

### Loki Validation

```bash
./tests/test-loki-validation.sh
```

Checks:
- Loki ready
- Push endpoint
- Query endpoint
- Labels available

## Destructive Tests

### Sleep/Wake Cycle

⚠️ **Warning:** Puts nodes to sleep

```bash
./tests/test-sleep-wake-cycle.sh
```

Requires confirmation. Tests:
1. Record cluster state
2. Trigger sleep
3. Send WoL packets
4. Measure wake time
5. Validate recovery

### Idempotency Test

```bash
./tests/test-idempotence.sh [cycles]
```

Runs multiple deployment cycles to verify idempotency.

## Writing Tests

### Test Structure

```bash
#!/bin/bash
# Test description
set -e

# Test logic
if condition; then
    echo "PASS: Test description"
else
    echo "FAIL: Test description"
    exit 1
fi
```

### Best Practices

1. Clear test names
2. One assertion per test
3. Clean up after tests
4. Exit codes (0=pass, 1=fail)
5. Descriptive output

### Example Test

```bash
#!/bin/bash
# Test: Verify Prometheus is healthy

set -e

echo "Testing Prometheus health..."

response=$(curl -s http://192.168.4.63:30090/-/healthy)

if [[ "$response" == *"Healthy"* ]]; then
    echo "PASS: Prometheus is healthy"
else
    echo "FAIL: Prometheus health check failed"
    exit 1
fi
```

## Test Output

### Success

```
✓ Pod Status - All pods Running and Ready
✓ Service Endpoints - All services have endpoints
✓ Health Endpoints - All health checks passed
```

### Failure

```
✗ Pod Status - loki-0 not Ready
  → Check: kubectl logs loki-0 -n monitoring
```

## Debugging Failed Tests

1. Check test output
2. Review pod logs
3. Run diagnostics
4. Check events

```bash
# After test failure
kubectl logs <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>
./scripts/diagnose-monitoring-stack.sh
```

## Related Documentation

- [Contributing](contributing.md)
- [Development Workflow](development-workflow.md)
- [Troubleshooting](../troubleshooting/README.md)
