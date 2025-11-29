# Reference

This section provides reference documentation for VMStation.

## Documents

- [CLI Reference](cli-reference.md) - Command-line reference
- [API Reference](api-reference.md) - API documentation
- [Configuration Reference](configuration-reference.md) - Configuration options
- [Glossary](glossary.md) - Terms and definitions

## Quick Reference

### Access URLs

| Service | URL |
|---------|-----|
| Grafana | http://192.168.4.63:30300 |
| Prometheus | http://192.168.4.63:30090 |
| Loki | http://192.168.4.63:31100 |
| Jellyfin | http://192.168.4.63:30096 |

### Node Information

| Node | IP | MAC |
|------|----|-----|
| masternode | 192.168.4.63 | - |
| storagenodet3500 | 192.168.4.61 | b8:ac:6f:7e:6c:9d |
| homelab | 192.168.4.62 | d0:94:66:30:d6:63 |

### Common Commands

```bash
# Cluster status
kubectl get nodes

# Pod status
kubectl get pods -A

# Wake nodes
wakeonlan b8:ac:6f:7e:6c:9d  # storagenodet3500
wakeonlan d0:94:66:30:d6:63  # homelab
```
