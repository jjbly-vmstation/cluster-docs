# Troubleshooting

This section provides guides for diagnosing and resolving issues in VMStation.

## Documents

- [Common Issues](common-issues.md) - Frequently encountered problems
- [Monitoring Issues](monitoring-issues.md) - Monitoring stack problems
- [Grafana Fixes](grafana-fixes.md) - Grafana-specific solutions
- [Storage Issues](storage-issues.md) - Storage troubleshooting
- [Network Issues](network-issues.md) - Network diagnostics
- [Diagnostic Procedures](diagnostic-procedures.md) - Tools and procedures

## Quick Diagnostics

### Automated Diagnostics

```bash
# Run diagnostics
./scripts/diagnose-monitoring-stack.sh

# Run validation
./scripts/validate-monitoring-stack.sh

# Run complete test suite
./tests/test-complete-validation.sh
```

### Common Commands

```bash
# Check nodes
kubectl get nodes

# Check pods
kubectl get pods -A | grep -v Running

# Check events
kubectl get events -A --sort-by='.lastTimestamp' | tail -20

# Check logs
kubectl logs <pod-name> -n <namespace>
```

## Quick Fixes

### Pod CrashLoopBackOff

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

### Node NotReady

```bash
kubectl describe node <node>
ssh <node> "systemctl status kubelet"
```

### PVC Pending

```bash
kubectl get sc
kubectl describe pvc <pvc> -n <namespace>
```

### Service Unreachable

```bash
kubectl get endpoints <service> -n <namespace>
kubectl get svc <service> -n <namespace>
```

## Getting Help

1. Check this troubleshooting guide
2. Run diagnostic scripts
3. Review pod logs
4. Check Grafana dashboards
5. Review deployment artifacts in `ansible/artifacts/`
