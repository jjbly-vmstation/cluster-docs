# Deployment

Deployment guides for VMStation components.

## Deployment Options

VMStation supports multiple deployment paths:

| Option | Description | Recommendation |
|--------|-------------|----------------|
| [Kubespray](kubespray-deployment.md) | Production-grade Kubernetes | **Recommended** |
| [Cluster Deployment](cluster-deployment.md) | kubeadm-based (legacy) | Debian-only |
| [Monitoring](monitoring-deployment.md) | Prometheus/Grafana/Loki | Required |
| [Infrastructure](infrastructure-services.md) | NTP, Syslog, Kerberos | Recommended |
| [Applications](application-deployment.md) | Jellyfin, etc. | Optional |

## Quick Deployment

### Recommended (Kubespray)

```bash
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray
./scripts/validate-monitoring-stack.sh
```

### Legacy (Debian kubeadm)

```bash
./deploy.sh reset
./deploy.sh setup
./deploy.sh debian
./deploy.sh monitoring
./deploy.sh infrastructure
```

## Deployment Guides

- [Cluster Deployment](cluster-deployment.md) - Core Kubernetes
- [Kubespray Deployment](kubespray-deployment.md) - Production deployment
- [Monitoring Deployment](monitoring-deployment.md) - Observability stack
- [Application Deployment](application-deployment.md) - Workloads
- [Infrastructure Services](infrastructure-services.md) - Core services

- [Identity Stack Fixes (2025-12-10)](identity-fixes-20251210.md) - Notes and operational guidance for recent identity playbook fixes
- [Diff-Patches: 20251210T175647Z](diff-patches-20251210T175647Z.md) - Patch index, scripts, manifests and automation for identity stack improvements

## Deployment Reports

Historical deployment reports:
- [Deployment Final Status](deployment-reports/deployment-final-status.md)
- [Deployment Report 2025-10-14](deployment-reports/deployment-report-20251014.md)

## Validation

After deployment:

```bash
# Validate monitoring
./scripts/validate-monitoring-stack.sh

# Complete validation
./tests/test-complete-validation.sh

# Check cluster
kubectl get nodes
kubectl get pods -A
```

## Related Documentation

- [Getting Started](../getting-started/README.md)
- [Architecture](../architecture/README.md)
- [Troubleshooting](../troubleshooting/README.md)
