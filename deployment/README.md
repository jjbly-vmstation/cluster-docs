# Deployment

This section covers all deployment aspects of VMStation, from initial cluster setup to application deployment.

## Documents

### Deployment Guides
- [Cluster Deployment](cluster-deployment.md) - Complete cluster deployment guide
- [Kubespray Deployment](kubespray-deployment.md) - Kubespray-specific deployment
- [Monitoring Deployment](monitoring-deployment.md) - Deploying the monitoring stack
- [Application Deployment](application-deployment.md) - Deploying applications
- [Infrastructure Services](infrastructure-services.md) - Core infrastructure deployment

### Deployment Reports
- [Final Status](deployment-reports/deployment-final-status.md) - Latest deployment status
- [Report 2025-10-14](deployment-reports/deployment-report-20251014.md) - Detailed deployment report

## Quick Reference

### Deploy Commands

```bash
# Full deployment (recommended)
./deploy.sh reset
./deploy.sh setup
./deploy.sh kubespray

# Legacy Debian-only
./deploy.sh debian
./deploy.sh monitoring
./deploy.sh infrastructure
```

### Validation Commands

```bash
# Validate deployment
./scripts/validate-monitoring-stack.sh
./tests/test-complete-validation.sh
```

## Deployment Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                    Deployment Workflow                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ┌─────────┐     ┌─────────┐     ┌──────────┐     ┌──────────┐ │
│   │  Reset  │ ──▶ │  Setup  │ ──▶ │  Deploy  │ ──▶ │ Validate │ │
│   └─────────┘     └─────────┘     └──────────┘     └──────────┘ │
│                                         │                        │
│                           ┌─────────────┴─────────────┐         │
│                           ▼                           ▼         │
│                    ┌──────────────┐          ┌──────────────┐   │
│                    │  Kubespray   │    OR    │ Debian/RKE2  │   │
│                    └──────────────┘          └──────────────┘   │
│                           │                           │         │
│                           ▼                           ▼         │
│                    ┌──────────────┐          ┌──────────────┐   │
│                    │  Monitoring  │          │  Monitoring  │   │
│                    └──────────────┘          └──────────────┘   │
│                           │                           │         │
│                           ▼                           ▼         │
│                    ┌───────────────────────────────────────┐    │
│                    │           Infrastructure              │    │
│                    └───────────────────────────────────────┘    │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

## Prerequisites Checklist

- [ ] All nodes powered on
- [ ] SSH connectivity verified
- [ ] Ansible installed
- [ ] Inventory configured
- [ ] Storage directories created

See [Prerequisites](../getting-started/prerequisites.md) for details.
