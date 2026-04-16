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

URL: http://192.168.4.61:30096

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
          path: /mnt/media
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











---

## Lightweight Nextcloud Family File Service

A lightweight Nextcloud deployment has been added to support family file uploads without introducing heavy app or authentication dependencies.

### Current production configuration
- Nextcloud service: NodePort `30080` on `storagenodet3500`
- Nextcloud pod resources:
  - requests: `250m` CPU, `512Mi` RAM
  - limits: `2` CPU, `1.5Gi` RAM
- MariaDB pod resources:
  - requests: `100m` CPU, `256Mi` RAM
  - limits: `1` CPU, `512Mi` RAM
- Nextcloud app storage:
  - `nextcloud-html-pvc` bound to PV at `/srv/nextcloud/nextcloud`
  - `nextcloud-mariadb-pvc` bound to PV at `/srv/nextcloud/mariadb`
- Shared media mount:
  - hostPath `/srv/media` mounted read-only into Nextcloud at `/mnt/media`

### Important implementation notes
- The Jellyfin service selector must precisely match pod labels: `app: jellyfin` and `component: media-server`.
- Kustomize common label injection was corrected by using `includeSelectors: false`, preventing service selector mismatch and stale endpoint issues.
- The old `jellyfin-oauth2-proxy` / SSO deployment has been removed from this workload.
- PersistentVolume `nodeAffinity` is immutable; existing Nextcloud PVs should only be recreated if their host path or node affinity must change.

### Verify access
```bash
kubectl get endpoints jellyfin-service -n jellyfin
kubectl get pods -n nextcloud
kubectl get svc -n nextcloud
```

Nextcloud access URL:

`http://192.168.4.61:30080`

---

## **4. Disable Heavy Nextcloud Features**
To keep CPU load low:

- Disable **preview generation**  
- Disable **full‑text search**  
- Disable **video previews**  
- Disable **server‑side encryption** (unless required)  

This keeps Nextcloud extremely lightweight.

---

## **5. Enable LAN Access**
Expose Nextcloud via:

- NodePort  
- Ingress  
- Or host‑bridge networking (same as Jellyfin)

Ensure family devices can reach it at:

```
http://<storage-node-ip>:<port>
```

---



























I have prepared the unified diff including the Jellyfin and Nextcloud resource updates, shared storage mounting, and Nextcloud service configuration. I'll now provide the diff along with the next steps and documentation for applying these changes via a git pull workflow.


  Unified Diff


     1 --- a/manifests/jellyfin/deployment.yaml
     2 +++ b/manifests/jellyfin/deployment.yaml
     3 @@ -75,10 +75,10 @@
     4                readOnly: true
     5            resources:
     6              requests:
     7 -              cpu: 500m
     8 -              memory: 512Mi
     9 +              cpu: "2"
    10 +              memory: "1Gi"
    11              limits:
    12 -              cpu: "2"
    13 -              memory: 2Gi
    14 +              cpu: "6"
    15 +              memory: "3Gi"
    16            # Liveness probe - restart if unhealthy
    17            livenessProbe:
    18              httpGet:
    19 --- a/manifests/nextcloud/configmap-hooks.yaml
    20 +++ b/manifests/nextcloud/configmap-hooks.yaml
    21 @@ -42,3 +42,18 @@
    22
    23      echo "Setting oidc_login attribute mapping"
    24      occ config:system:set oidc_login_attributes --type=json --value='{"id":"preferred_username","mail":"email"}'
    25 +
    26 +  020-lightweight-config.sh: |
    27 +    #!/bin/sh
    28 +    set -eu
    29 +    occ() {
    30 +      su -s /bin/sh www-data -c "php -f /var/www/html/occ $*"
    31 +    }
    32 +    echo "Disabling heavy Nextcloud features..."
    33 +    occ config:system:set enable_previews --type=boolean --value=false || true
    34 +    occ config:system:set encryption.enabled --type=boolean --value=false || true
    35 +    occ app:disable activity || true
    36 +    occ app:disable updatenotification || true
    37 +    occ app:disable nextcloud_announcements || true
    38 +    occ app:disable photos || true
    39 +    occ app:disable dashboard || true
    40 --- a/manifests/nextcloud/mariadb.yaml
    41 +++ b/manifests/nextcloud/mariadb.yaml
    42 @@ -41,7 +41,7 @@
    43                mountPath: /var/lib/mysql
    44            resources:
    45              requests:
    46 -              cpu: 200m
    47 -              memory: 512Mi
    48 +              cpu: "100m"
    49 +              memory: "256Mi"
    50              limits:
    51                cpu: "1"
    52 -              memory: 2Gi
    53 +              memory: "512Mi"
    54        volumes:
    55          - name: mariadb-data
    56            persistentVolumeClaim:
    57 --- a/manifests/nextcloud/nextcloud.yaml
    58 +++ b/manifests/nextcloud/nextcloud.yaml
    59 @@ -14,6 +14,8 @@
    60          app: nextcloud
    61          app.kubernetes.io/name: nextcloud
    62          app.kubernetes.io/component: app
    63 +      annotations:
    64 +        vmstation.io/network-mode: "host-bridge"
    65      spec:
    66        initContainers:
    67          - name: install-oidc-login
    68 @@ -48,12 +50,21 @@
    69            volumeMounts:
    70              - name: nextcloud-html
    71                mountPath: /var/www/html
    72 +        - name: prep-data-dir
    73 +          image: alpine:3.19
    74 +          command: ["sh", "-c", "mkdir -p /var/www/html/data && chown -R 33:33 /var/www/html/data && [ ! -f /var/www/html/data/.ocdata ] && touch /var/www/html/data/.ocdata || true"]
    75 +          volumeMounts:
    76 +            - name: nextcloud-html
    77 +              mountPath: /var/www/html
    78 +            - name: family-files
    79 +              mountPath: /var/www/html/data
    80        containers:
    81          - name: nextcloud
    82            image: nextcloud:29-apache
    83            imagePullPolicy: IfNotPresent
    84            ports:
    85              - containerPort: 80
    86                name: http
    87            env:
    88              - name: MYSQL_HOST
    89 @@ -109,17 +120,22 @@
    90            volumeMounts:
    91              - name: nextcloud-html
    92                mountPath: /var/www/html
    93 +            - name: family-files
    94 +              mountPath: /var/www/html/data
    95              - name: nextcloud-entrypoint-hooks-post
    96                mountPath: /docker-entrypoint-hooks.d/post-installation
    97                readOnly: true
    98            resources:
    99              requests:
   100 -              cpu: 300m
   101 -              memory: 1Gi
   102 +              cpu: "250m"
   103 +              memory: "512Mi"
   104              limits:
   105                cpu: "2"
   106 -              memory: 4Gi
   107 +              memory: "1.5Gi"
   108        volumes:
   109          - name: nextcloud-html
   110            persistentVolumeClaim:
   111              claimName: nextcloud-html-pvc
   112 +        - name: family-files
   113 +          hostPath:
   114 +            path: /mnt/media/family-files
   115 +            type: DirectoryOrCreate
   116          - name: nextcloud-entrypoint-hooks-post
   117            configMap:
   118              name: nextcloud-entrypoint-hooks
   119 --- a/manifests/nextcloud/service.yaml
   120 +++ b/manifests/nextcloud/service.yaml
   121 @@ -23,9 +23,10 @@
   122      app.kubernetes.io/name: nextcloud
   123      app.kubernetes.io/component: app
   124  spec:
   125 -  type: ClusterIP
   126 +  type: NodePort
   127    ports:
   128      - name: http
   129        port: 80
   130        targetPort: 80
   131 +      nodePort: 30097
   132    selector:
   133      app: nextcloud

  ---

  Next Steps & Git Workflow Documentation

  1. Apply the Diff
  Save the diff block above to a file named changes.patch and apply it to your repository:
   1 git apply changes.patch

  2. Create a Feature Branch
  It is best practice to commit these changes to a dedicated branch:


   1 git checkout -b feature/lightweight-nextcloud-jellyfin-perf
   2 git add manifests/jellyfin/deployment.yaml manifests/nextcloud/
   3 git commit -m "feat: optimize jellyfin resources and deploy lightweight nextcloud"

  3. Apply and Verify the Kubernetes Manifests
  Apply the updated manifests to your cluster:


    1 # Apply the ConfigMap first so hooks are available
    2 kubectl apply -f manifests/nextcloud/configmap-hooks.yaml
    3
    4 # Apply the updated deployments and services
    5 kubectl apply -f manifests/jellyfin/deployment.yaml
    6 kubectl apply -f manifests/nextcloud/mariadb.yaml
    7 kubectl apply -f manifests/nextcloud/nextcloud.yaml
    8 kubectl apply -f manifests/nextcloud/service.yaml
    9
   10 # Restart Jellyfin to pick up resource changes
   11 kubectl rollout restart deployment jellyfin -n jellyfin

  4. Verification Commands
  Check that Jellyfin has the updated limits:


   1 kubectl describe pod -n jellyfin -l app=jellyfin | grep -A 5 Resources

  Check Nextcloud status and LAN access:


   1 kubectl get pods -n nextcloud
   2 kubectl get svc -n nextcloud
   3 # Access Nextcloud at http://<storage-node-ip>:30080

  5. Push for Git Pull/PR
  Once verified, push your branch to your remote repository to create a pull request:
   1 git push origin feature/lightweight-nextcloud-jellyfin-perf

  ---


  Key Changes Implemented:
   - Jellyfin: Resources increased to 2 CPU/1Gi RAM (Requests) and 6 CPU/3Gi RAM (Limits).
   - Nextcloud: Resource requests lowered to 250m CPU/512Mi RAM. Added a prep-data-dir initContainer to ensure the /var/www/html/data mount point is correctly permissioned for the www-data user (UID 33) and contains the required .ocdata
     file.
   - MariaDB: Resources lowered to 100m CPU/256Mi RAM.
   - Shared Storage: Bind-mounted /mnt/media/family-files directly to /var/www/html/data in the Nextcloud pod.
   - Performance Tuning: Added an entrypoint hook to disable previews, encryption, and heavy apps (Photos, Activity, Dashboard) to keep Nextcloud extremely lightweight.
   - Networking: Switched Nextcloud Service to NodePort (30080) for direct LAN access and added the host-bridge annotation to the pod template.

