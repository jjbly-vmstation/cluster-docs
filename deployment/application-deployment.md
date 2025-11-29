# Application Deployment

Deploy applications on VMStation.

## Overview

VMStation supports deploying various workloads. This guide covers application deployment patterns.

## Jellyfin

Jellyfin media server is deployed automatically with the monitoring stack.

### Deployment

```bash
kubectl apply -f manifests/apps/jellyfin.yaml
```

### Configuration

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
        - name: media
          mountPath: /media
      volumes:
      - name: config
        hostPath:
          path: /srv/jellyfin/config
      - name: media
        hostPath:
          path: /media
---
apiVersion: v1
kind: Service
metadata:
  name: jellyfin
spec:
  type: NodePort
  selector:
    app: jellyfin
  ports:
  - port: 8096
    targetPort: 8096
    nodePort: 30096
```

### Access

URL: http://192.168.4.63:30096

## Deployment Patterns

### NodeSelector

Schedule pods on specific nodes:

```yaml
spec:
  nodeSelector:
    kubernetes.io/hostname: storagenodet3500
```

### Resource Limits

Define resource constraints:

```yaml
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

### Persistent Storage

Use PVC for data persistence:

```yaml
spec:
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data
  containers:
  - name: app
    volumeMounts:
    - name: data
      mountPath: /data
```

### NodePort Services

Expose services on fixed ports:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

## Auto-Sleep Considerations

For pods on auto-sleep nodes (storagenodet3500, homelab):

### Tolerate Node Not Ready

```yaml
spec:
  tolerations:
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoSchedule"
    tolerationSeconds: 300
```

### Schedule on Always-On Node

```yaml
spec:
  nodeSelector:
    kubernetes.io/hostname: masternode
```

## Deploy Custom Application

### Create Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:latest
        ports:
        - containerPort: 8080
```

### Apply

```bash
kubectl apply -f myapp.yaml
```

### Verify

```bash
kubectl get pods -l app=myapp
kubectl get svc myapp
```

## Troubleshooting

### Pod Pending

```bash
kubectl describe pod <pod-name>
```

Common causes:
- Node not available (sleeping)
- Insufficient resources
- PVC pending

### Image Pull Errors

```bash
kubectl describe pod <pod-name> | grep -A5 Events
```

Check:
- Image name/tag
- Registry access
- Image pull secrets

### CrashLoopBackOff

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

## Related Documentation

- [Cluster Deployment](cluster-deployment.md)
- [Monitoring Deployment](monitoring-deployment.md)
- [Jellyfin](../components/applications/jellyfin.md)
