# Jellyfin

Jellyfin is the media streaming application deployed on VMStation.

## Overview

| Property | Value |
|----------|-------|
| Port | 8096 (internal), 30096 (NodePort) |
| Node | storagenodet3500 |
| URL | http://192.168.4.63:30096 |

## Deployment

### Manifest

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
---
apiVersion: v1
kind: Service
metadata:
  name: jellyfin
  namespace: default
spec:
  type: NodePort
  selector:
    app: jellyfin
  ports:
  - port: 8096
    targetPort: 8096
    nodePort: 30096
```

### Deploy

```bash
kubectl apply -f manifests/apps/jellyfin.yaml
```

## Configuration

### Media Directories

On storagenodet3500:

```bash
sudo mkdir -p /media/{movies,tv,music}
sudo chown -R 1000:1000 /media
```

### First-Time Setup

1. Open http://192.168.4.63:30096
2. Complete setup wizard
3. Add media libraries
4. Configure users

## Storage

### Directory Structure

```
/media/
├── movies/
├── tv/
├── music/
└── photos/
```

### Permissions

```bash
sudo chown -R 1000:1000 /media
sudo chmod -R 755 /media
```

## Troubleshooting

### Pod Not Scheduling

Check node availability:
```bash
kubectl get nodes
kubectl describe node storagenodet3500
```

Wake node if sleeping:
```bash
wakeonlan b8:ac:6f:7e:6c:9d
```

### Can't Access Media

Check volume mounts:
```bash
kubectl describe pod -l app=jellyfin
```

### Web UI Not Loading

Check service:
```bash
kubectl get svc jellyfin
curl http://192.168.4.63:30096
```

## Related Documentation

- [Application Deployment](../../deployment/application-deployment.md)
- [Storage Architecture](../../architecture/storage-architecture.md)
