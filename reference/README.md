# Reference

Reference documentation for VMStation.

## Documents

- [CLI Reference](cli-reference.md) - Command line tools
- [API Reference](api-reference.md) - API endpoints
- [Configuration Reference](configuration-reference.md) - Configuration options
- [Glossary](glossary.md) - Terminology

## Quick Reference

### Commands

```bash
# Deployment
./deploy.sh debian         # Deploy Debian cluster
./deploy.sh kubespray      # Deploy Kubespray cluster
./deploy.sh monitoring     # Deploy monitoring
./deploy.sh infrastructure # Deploy infrastructure
./deploy.sh reset          # Reset cluster

# Validation
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh

# Diagnostics
./scripts/diagnose-monitoring-stack.sh
./scripts/remediate-monitoring-stack.sh
```

### Service URLs

| Service | URL |
|---------|-----|
| Grafana | http://192.168.4.63:30300 |
| Prometheus | http://192.168.4.63:30090 |
| Loki | http://192.168.4.63:31100 |
| Jellyfin | http://192.168.4.63:30096 |

### kubectl

```bash
# Use with admin config
kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes

# Or set KUBECONFIG
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
```

## Related Documentation

- [Getting Started](../getting-started/README.md)
- [Operations](../operations/README.md)
- [Troubleshooting](../troubleshooting/README.md)
