# Jellyfin

Media streaming server.

## Overview

Jellyfin provides media streaming for the homelab.

| Property | Value |
|----------|-------|
| Port | 8096 (internal), 30096 (NodePort) |
| Node | storagenodet3500 |
| Storage | Host path mount |

## Access

URL: http://192.168.4.63:30096

## Deployment

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

## Storage

### Config Directory

```
/srv/jellyfin/config/
```

Contains Jellyfin configuration and database.

### Media Directory

```
/media/
```

Mount point for media files.

## Auto-Sleep Consideration

Jellyfin runs on storagenodet3500 which has auto-sleep enabled.

### Wake on Access

When accessing Jellyfin:
1. Request fails (node sleeping)
2. WoL packet sent
3. Node wakes
4. Jellyfin becomes available
5. Retry access

### Monitor for Idle

Auto-sleep monitors Jellyfin activity before sleeping node.

## Troubleshooting

### Pod Pending

Node may be sleeping:

```bash
# Check node status
kubectl get nodes

# Wake node
wakeonlan b8:ac:6f:7e:6c:9d
```

### Container Crashing

Check logs:

```bash
kubectl logs -l app=jellyfin
```

Check storage mounts:

```bash
ssh storagenodet3500 "ls -la /srv/jellyfin/config"
ssh storagenodet3500 "ls -la /media"
```

### Permission Issues

```bash
ssh storagenodet3500 "chown -R 1000:1000 /srv/jellyfin"
```

## Backup

### Config Backup

```bash
ssh storagenodet3500 "tar -czf /backup/jellyfin-config.tar.gz /srv/jellyfin/config"
```

### Database Location

```
/srv/jellyfin/config/data/jellyfin.db
```

## Related Documentation

- [Application Deployment](../../deployment/application-deployment.md)
- [Power Management](../../operations/power-management.md)
