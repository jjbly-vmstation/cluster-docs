# Diagnostic Procedures

This guide covers diagnostic tools and procedures for VMStation.

## Automated Diagnostics

### Monitoring Stack Diagnostics

```bash
./scripts/diagnose-monitoring-stack.sh
```

**What it collects:**
- Pod status and logs
- Service and endpoint configurations
- PVC/PV bindings
- ConfigMaps
- Host directory permissions
- Readiness probe results

**Output:**
- Timestamped directory in `/tmp/monitoring-diagnostics-*`
- Analysis file: `00-ANALYSIS-AND-RECOMMENDATIONS.txt`

### Monitoring Validation

```bash
./scripts/validate-monitoring-stack.sh
```

**Tests:**
1. Pod Status - All Running and Ready
2. Service Endpoints - Populated
3. PVC/PV Bindings - Bound
4. Health Endpoints - HTTP checks
5. DNS Resolution - Service names
6. Container Restarts - Stability
7. Log Analysis - Error scanning

### Complete Validation Suite

```bash
./tests/test-complete-validation.sh
```

**Phases:**
1. Configuration validation
2. Monitoring health
3. Sleep/wake cycle (optional)

## Manual Diagnostics

### Cluster Health

```bash
# Node status
kubectl get nodes -o wide

# All pods status
kubectl get pods -A

# Recent events
kubectl get events -A --sort-by='.lastTimestamp' | tail -20
```

### Pod Diagnostics

```bash
# Pod details
kubectl describe pod <pod> -n <namespace>

# Current logs
kubectl logs <pod> -n <namespace>

# Previous logs (after crash)
kubectl logs <pod> -n <namespace> --previous

# Multi-container logs
kubectl logs <pod> -n <namespace> -c <container>

# Follow logs
kubectl logs -f <pod> -n <namespace>
```

### Service Diagnostics

```bash
# Service details
kubectl describe svc <service> -n <namespace>

# Endpoints
kubectl get endpoints <service> -n <namespace>

# Port forward for testing
kubectl port-forward svc/<service> 8080:80 -n <namespace>
```

### Storage Diagnostics

```bash
# PVC status
kubectl get pvc -n <namespace>

# PV details
kubectl describe pv <pv-name>

# Storage class
kubectl get sc

# Local provisioner
kubectl get pods -n local-path-storage
```

### Network Diagnostics

```bash
# DNS test
kubectl run test --rm -it --image=busybox -- nslookup kubernetes

# Connectivity test
kubectl run test --rm -it --image=busybox -- wget -O- http://prometheus.monitoring:9090/-/healthy

# CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns
```

## Health Checks

### Prometheus

```bash
# Health endpoint
curl http://192.168.4.63:30090/-/healthy

# Ready endpoint
curl http://192.168.4.63:30090/-/ready

# Targets status
curl -s http://192.168.4.63:30090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, health: .health}'
```

### Loki

```bash
# Ready endpoint
curl http://192.168.4.63:31100/ready

# Labels (verify data)
curl -s http://192.168.4.63:31100/loki/api/v1/labels | jq
```

### Grafana

```bash
# Health endpoint
curl http://192.168.4.63:30300/api/health

# Datasources
curl -s http://192.168.4.63:30300/api/datasources | jq '.[].name'
```

## Log Collection

### Collect All Monitoring Logs

```bash
mkdir -p /tmp/monitoring-logs

# Prometheus
kubectl logs prometheus-0 -n monitoring -c prometheus > /tmp/monitoring-logs/prometheus.log
kubectl logs prometheus-0 -n monitoring -c prometheus-config-reloader > /tmp/monitoring-logs/prometheus-reloader.log 2>&1 || true

# Loki
kubectl logs loki-0 -n monitoring > /tmp/monitoring-logs/loki.log

# Grafana
kubectl logs -n monitoring -l app=grafana > /tmp/monitoring-logs/grafana.log

# Promtail
kubectl logs -n monitoring -l app=promtail --all-containers > /tmp/monitoring-logs/promtail.log
```

### Collect Node Logs

```bash
ssh <node> "journalctl -u kubelet -n 100" > /tmp/kubelet-<node>.log
ssh <node> "journalctl -u containerd -n 100" > /tmp/containerd-<node>.log
```

### Collect Wake Logs

```bash
./scripts/vmstation-collect-wake-logs.sh
```

## Resource Analysis

### CPU/Memory Usage

```bash
# Node resources
kubectl top nodes

# Pod resources
kubectl top pods -n monitoring

# Detailed pod resources
kubectl describe pod <pod> -n <namespace> | grep -A5 "Requests\|Limits"
```

### Disk Usage

```bash
# Node disk
df -h

# Monitoring data
du -sh /srv/monitoring_data/*

# PVC directories
du -sh /srv/monitoring_data/pvc-*
```

## Remediation Scripts

### Automated Remediation

```bash
./scripts/remediate-monitoring-stack.sh
```

**Actions:**
1. Backup current state
2. Patch SecurityContexts
3. Fix ConfigMaps
4. Fix permissions
5. Restart pods
6. Validate

### Manual Remediation Steps

```bash
# Fix Prometheus permissions
sudo chown -R 65534:65534 /srv/monitoring_data/prometheus

# Fix Loki permissions
sudo chown -R 10001:10001 /srv/monitoring_data/loki

# Restart monitoring pods
kubectl delete pod prometheus-0 loki-0 -n monitoring
kubectl rollout restart deployment/grafana -n monitoring
```

## Debugging Commands

### Exec into Pod

```bash
# Shell access
kubectl exec -it <pod> -n <namespace> -- /bin/sh

# Specific container
kubectl exec -it <pod> -n <namespace> -c <container> -- /bin/sh
```

### Network Debugging

```bash
# Create debug pod
kubectl run netdebug --rm -it --image=nicolaka/netshoot -- /bin/bash

# Inside debug pod:
# dig kubernetes.default.svc.cluster.local
# curl http://prometheus.monitoring:9090/-/healthy
# nslookup loki.monitoring
```

### View Raw Resources

```bash
# Pod YAML
kubectl get pod <pod> -n <namespace> -o yaml

# Events for resource
kubectl get events --field-selector involvedObject.name=<name> -n <namespace>
```

## Diagnostic Checklist

### Quick Checks

- [ ] All nodes Ready: `kubectl get nodes`
- [ ] All pods Running: `kubectl get pods -A | grep -v Running`
- [ ] Prometheus healthy: `curl http://192.168.4.63:30090/-/healthy`
- [ ] Loki ready: `curl http://192.168.4.63:31100/ready`
- [ ] Grafana healthy: `curl http://192.168.4.63:30300/api/health`

### Deep Checks

- [ ] No recent restarts: `kubectl get pods -n monitoring -o wide`
- [ ] Endpoints populated: `kubectl get endpoints -n monitoring`
- [ ] PVCs bound: `kubectl get pvc -n monitoring`
- [ ] No error logs: Review pod logs
- [ ] Storage available: `df -h /srv/monitoring_data`

## Related Documentation

- [Common Issues](common-issues.md)
- [Monitoring Issues](monitoring-issues.md)
- [Day-2 Operations](../operations/day-2-operations.md)
