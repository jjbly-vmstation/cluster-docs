# Application Deployment

This guide covers deploying applications on VMStation.

## Deployment Methods

1. **Kubernetes Manifests** - YAML files
2. **Helm Charts** - Package manager
3. **Ansible Playbooks** - Automated deployment

## Quick Start

### Deploy from Manifest

```bash
kubectl apply -f manifests/apps/<app>.yaml
```

### Verify Deployment

```bash
kubectl get pods -n <namespace>
kubectl get svc -n <namespace>
```

## Jellyfin Media Server

### Deploy

```bash
kubectl apply -f manifests/apps/jellyfin.yaml
```

### Access

- URL: http://192.168.4.63:30096
- Complete setup wizard on first access

### Configuration

Jellyfin is deployed on storagenodet3500 for storage access.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jellyfin
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jellyfin
  template:
    metadata:
      labels:
        app: jellyfin
    spec:
      nodeSelector:
        kubernetes.io/hostname: storagenodet3500
      containers:
      - name: jellyfin
        image: jellyfin/jellyfin:latest
        ports:
        - containerPort: 8096
        volumeMounts:
        - name: config
          mountPath: /config
        - name: cache
          mountPath: /cache
        - name: media
          mountPath: /media
      volumes:
      - name: config
        hostPath:
          path: /srv/jellyfin/config
      - name: cache
        hostPath:
          path: /srv/jellyfin/cache
      - name: media
        hostPath:
          path: /media
```

### Media Storage

```bash
# On storagenodet3500
sudo mkdir -p /media/{movies,tv,music}
sudo chown -R 1000:1000 /media
```

## Creating Custom Applications

### Basic Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
  namespace: default
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080
```

### With Persistent Storage

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-data
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: my-app-data
```

### With ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
  namespace: default
data:
  config.yaml: |
    setting1: value1
    setting2: value2
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app
        volumeMounts:
        - name: config
          mountPath: /etc/myapp
      volumes:
      - name: config
        configMap:
          name: my-app-config
```

## Node Scheduling

### Schedule on Specific Node

```yaml
spec:
  nodeSelector:
    kubernetes.io/hostname: storagenodet3500
```

### Avoid Control Plane

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-role.kubernetes.io/control-plane
            operator: DoesNotExist
```

### Tolerate Auto-Sleep

For apps that can handle node sleep:

```yaml
spec:
  tolerations:
  - key: "node.kubernetes.io/unreachable"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300
```

## Monitoring Applications

### Add Prometheus Scraping

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

### View Application Logs

```bash
# View logs
kubectl logs -f deployment/my-app

# In Grafana (Loki)
{namespace="default", app="my-app"}
```

## Application Management

### Scale Deployment

```bash
kubectl scale deployment my-app --replicas=3
```

### Update Image

```bash
kubectl set image deployment/my-app my-app=my-app:v2
```

### Rollback

```bash
kubectl rollout undo deployment/my-app
```

### Delete Application

```bash
kubectl delete -f manifests/apps/my-app.yaml
```

## Best Practices

### Resource Limits

Always set resource limits:

```yaml
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"
  requests:
    memory: "256Mi"
    cpu: "250m"
```

### Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Security Context

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
```

## Troubleshooting

### Pod Not Starting

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Service Not Accessible

```bash
kubectl get endpoints <service-name>
curl http://<node-ip>:<nodeport>
```

### Storage Issues

```bash
kubectl get pvc
kubectl describe pvc <pvc-name>
```

## Related Documentation

- [Cluster Deployment](cluster-deployment.md)
- [Storage Architecture](../architecture/storage-architecture.md)
- [Troubleshooting](../troubleshooting/common-issues.md)
