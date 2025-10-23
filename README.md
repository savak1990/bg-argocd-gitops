# BoardGame Shop - ArgoCD GitOps Repository (Development)

This repository contains the GitOps configuration for the BoardGame Shop Kubernetes development infrastructure, managed by ArgoCD.

## Repository Structure

```
.
├── README.md
├── cluster-addons/           # Infrastructure and cluster-level components
│   └── kube-prometheus-stack/
│       ├── Chart.yaml
│       └── values.yaml
└── applications/             # ArgoCD Application manifests
    ├── cluster-addons/
    │   ├── project.yaml
    │   └── kube-prometheus-stack.yaml
    └── boardgame-shop/       # Future application deployments
```

## Directory Descriptions

### `cluster-addons/`
Contains Helm charts and configurations for cluster-level infrastructure components:
- **kube-prometheus-stack**: Complete monitoring stack with Prometheus, Grafana, and AlertManager

### `applications/`
Contains ArgoCD Application and AppProject manifests that define how components are deployed:
- **cluster-addons/**: Applications for infrastructure components
- **boardgame-shop/**: Applications for the main BoardGame Shop services (future)

## Current Components

### Kube-Prometheus-Stack
A comprehensive monitoring solution that includes:
- **Prometheus**: Metrics collection and storage (7 days retention, 5Gi storage)
- **Grafana**: Visualization and dashboards with development-friendly access
- **AlertManager**: Alert routing and management
- **Node Exporter**: Node-level metrics
- **Kube State Metrics**: Kubernetes object metrics

#### Configuration
- Configuration file: `cluster-addons/kube-prometheus-stack/values.yaml`
- Optimized for development with reasonable resource limits
- Includes optional ingress configuration for easy access

## Getting Started

### Prerequisites
- ArgoCD installed and configured in your cluster
- Access to the cluster with appropriate RBAC permissions
- (Optional) Ingress controller for Grafana access

### Deploying the Monitoring Stack

1. Apply the cluster-addons project:
   ```bash
   kubectl apply -f applications/cluster-addons/project.yaml
   ```

2. Deploy kube-prometheus-stack:
   ```bash
   kubectl apply -f applications/cluster-addons/kube-prometheus-stack.yaml
   ```

The application will automatically sync and deploy the monitoring stack to the `monitoring` namespace.

## Accessing Services

### Grafana
- **Port Forward**: `kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring`
- **Ingress** (if enabled): http://grafana.dev.local
- **Default Credentials**: admin / dev-admin-password

### Prometheus
- **Port Forward**: `kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring`

### AlertManager
- **Port Forward**: `kubectl port-forward svc/kube-prometheus-stack-alertmanager 9093:9093 -n monitoring`

## Configuration Details

### Resource Allocation
| Component | CPU Request | CPU Limit | Memory Request | Memory Limit |
|-----------|-------------|-----------|----------------|--------------|
| **Prometheus** | 250m | 500m | 512Mi | 1Gi |
| **Grafana** | 100m | 200m | 128Mi | 256Mi |
| **AlertManager** | 50m | 100m | 64Mi | 128Mi |

### Storage
- **Prometheus**: 5Gi persistent storage, 7 days retention
- **Grafana**: 1Gi persistent storage for dashboards
- **AlertManager**: 500Mi persistent storage

## Adding New Components

### For Cluster Addons
1. Create a new directory under `cluster-addons/`
2. Add Helm chart files (`Chart.yaml`, `values.yaml`)
3. Create ArgoCD Application manifest in `applications/cluster-addons/`

### For Applications
1. Create configurations under `applications/boardgame-shop/`
2. Follow the same pattern as cluster-addons

## Customization

### Modifying Monitoring Configuration
Edit `cluster-addons/kube-prometheus-stack/values.yaml` to customize:
- Resource limits and requests
- Storage sizes and classes
- Grafana admin password
- Ingress settings
- Retention policies

### Adding Custom Dashboards
Grafana dashboards can be added via:
- Grafana UI (will persist in storage)
- ConfigMaps with dashboard JSON
- Grafana sidecar for automated dashboard loading

## Troubleshooting

### Common Issues

1. **Application Not Syncing**
   ```bash
   # Check ArgoCD application status
   kubectl get applications -n argocd
   
   # View application details
   kubectl describe application kube-prometheus-stack -n argocd
   ```

2. **Storage Issues**
   ```bash
   # Check PVC status
   kubectl get pvc -n monitoring
   
   # Verify storage class exists
   kubectl get storageclass
   ```

3. **Access Issues**
   ```bash
   # Check services
   kubectl get svc -n monitoring
   
   # Check ingress (if enabled)
   kubectl get ingress -n monitoring
   ```

### Useful Commands

```bash
# Check all monitoring pods
kubectl get pods -n monitoring

# View Prometheus configuration
kubectl get prometheus -n monitoring -o yaml

# Check Grafana logs
kubectl logs -n monitoring deployment/kube-prometheus-stack-grafana

# Force sync application
kubectl patch application kube-prometheus-stack -n argocd -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}' --type merge
```

## Security Considerations

- Default Grafana password should be changed for any exposed environment
- Consider using Kubernetes secrets for sensitive configuration
- Ingress should use TLS in any internet-facing deployment
- RBAC is configured through the AppProject for proper access controls

## Future Enhancements

- [ ] Add custom alert rules for BoardGame Shop applications
- [ ] Configure additional data sources (if needed)
- [ ] Add custom Grafana dashboards for application metrics
- [ ] Implement log aggregation with Loki (optional)
- [ ] Add service monitors for BoardGame Shop services