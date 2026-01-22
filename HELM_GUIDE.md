# Helm Deployment Guide

Complete guide for deploying the Ethereum node using Helm.

## Quick Start

### Deploy

```bash
# Generate JWT secret
./scripts/generate-jwt.sh

# Deploy with Helm
./scripts/helm-deploy.sh
```

### Verify

```bash
# Check deployment
helm list -A
kubectl get pods -n ethereum
kubectl get pods -n monitoring

# View logs
kubectl logs -f statefulset/geth -n ethereum
kubectl logs -f statefulset/prysm-beacon -n ethereum
```

### Access Grafana

```bash
kubectl port-forward -n monitoring svc/grafana 3000:3000
# Open: http://localhost:3000 (admin/admin)
```

## Configuration

### Default Storage (~75GB)

```yaml
Geth (light mode):    20GB
Prysm (beacon):       40GB
Prometheus:           10GB
Grafana:               5GB
```

### Custom Configuration

Edit `helm/ethereum-node/values.yaml` or create a custom values file:

```yaml
# custom-values.yaml
geth:
  storage:
    size: 50Gi
  resources:
    limits:
      memory: "8Gi"
      cpu: "2000m"

prysm:
  storage:
    size: 60Gi
```

Deploy with custom values:
```bash
helm install ethereum-node ./helm/ethereum-node \
  --values custom-values.yaml \
  --set jwt.secret="$(cat jwt.hex)"
```

## Management

### Upgrade

```bash
helm upgrade ethereum-node ./helm/ethereum-node \
  --set jwt.secret="$(cat jwt.hex)" \
  --reuse-values
```

### Rollback

```bash
helm rollback ethereum-node
```

### Uninstall

```bash
./scripts/helm-cleanup.sh
# or
helm uninstall ethereum-node
kubectl delete namespace ethereum monitoring
```

## Chart Structure

```
helm/ethereum-node/
├── Chart.yaml              # Chart metadata
├── values.yaml             # Configuration
├── README.md               # Chart documentation
└── templates/              # Kubernetes templates
    ├── _helpers.tpl
    ├── namespaces.yaml
    ├── jwt-secret.yaml
    ├── geth-*.yaml
    ├── prysm-*.yaml
    ├── prometheus-*.yaml
    ├── grafana-*.yaml
    └── peer-monitor-*.yaml
```

## Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.network` | Ethereum network | `mainnet` |
| `geth.syncMode` | Sync mode | `light` |
| `geth.storage.size` | Geth storage | `20Gi` |
| `prysm.storage.size` | Prysm storage | `40Gi` |
| `monitoring.enabled` | Enable monitoring | `true` |

## Common Commands

```bash
# List releases
helm list -A

# Check status
helm status ethereum-node

# View deployed values
helm get values ethereum-node

# Check pods
kubectl get pods -n ethereum -n monitoring

# Port forward Grafana
kubectl port-forward -n monitoring svc/grafana 3000:3000

# Port forward Geth RPC
kubectl port-forward -n ethereum svc/geth 8545:8545
```

## Troubleshooting

### Validate Chart

```bash
helm lint ./helm/ethereum-node
```

### Dry Run

```bash
helm install ethereum-node ./helm/ethereum-node \
  --set jwt.secret="test" \
  --dry-run --debug
```

### Check Logs

```bash
kubectl logs -f statefulset/geth -n ethereum
kubectl logs -f statefulset/prysm-beacon -n ethereum
kubectl get events -n ethereum --sort-by='.lastTimestamp'
```

### Storage Issues

```bash
kubectl get pvc -n ethereum
kubectl describe pvc geth-data-geth-0 -n ethereum
```

## Benefits of Helm

- ✅ Parameterized configuration
- ✅ Easy upgrades and rollbacks
- ✅ Version control
- ✅ Template validation
- ✅ Environment management

## Related Documentation

- [Chart README](./helm/ethereum-node/README.md) - Detailed chart docs
- [Main README](./README.md) - Project overview
- [Storage Guide](./STORAGE_GUIDE.md) - Storage planning
