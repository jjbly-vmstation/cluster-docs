# Troubleshooting

Troubleshooting guides for VMStation.

## Quick Diagnostics

```bash
# Run diagnostics
./scripts/diagnose-monitoring-stack.sh

# Run validation
./scripts/validate-monitoring-stack.sh

# Complete test suite
./tests/test-complete-validation.sh
```

## Documents

- [Common Issues](common-issues.md) - Frequently encountered problems
- [Monitoring Issues](monitoring-issues.md) - Prometheus, Grafana, Loki
- [Grafana Fixes](grafana-fixes.md) - Grafana-specific issues
- [Storage Issues](storage-issues.md) - PVC and volume problems
- [Network Issues](network-issues.md) - DNS and connectivity
- [Diagnostic Procedures](diagnostic-procedures.md) - Systematic debugging

## Quick Reference

### Check Cluster Health

```bash
kubectl get nodes
kubectl get pods -A | grep -v Running
kubectl get events --sort-by='.lastTimestamp' | tail -20
```

### Check Monitoring

```bash
kubectl get pods -n monitoring
curl http://192.168.4.63:30090/-/healthy
curl http://192.168.4.63:31100/ready
```

### Check Logs

```bash
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

## Common Quick Fixes

### Pod CrashLoopBackOff

```bash
# Check logs
kubectl logs <pod> -n <namespace>

# Check events
kubectl describe pod <pod> -n <namespace>
```

### Node NotReady

```bash
# Check kubelet
ssh <node> "systemctl status kubelet"
ssh <node> "journalctl -xeu kubelet -n 50"
```

### DNS Not Working

```bash
# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### PVC Pending

```bash
# Check storage class
kubectl get sc

# Check provisioner
kubectl get pods -n local-path-storage
```

## Related Documentation

- [Operations](../operations/README.md)
- [Deployment](../deployment/README.md)
- [Architecture](../architecture/README.md)
