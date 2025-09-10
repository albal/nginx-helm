# nginx-helm

A Helm chart for deploying nginx web server in Kubernetes with LoadBalancer service support.

## Overview

This Helm chart deploys nginx in a Kubernetes cluster with the following features:

- **High Availability**: Deploys 2 nginx pod replicas by default
- **LoadBalancer Service**: Exposes nginx through a LoadBalancer service for external access
- **Production Ready**: Includes resource limits, liveness/readiness probes, and proper security contexts
- **Highly Configurable**: Comprehensive values.yaml with sensible defaults
- **Kubernetes Best Practices**: Follows Helm and Kubernetes recommended practices

## Prerequisites

- Kubernetes cluster v1.19+
- Helm 3.0+
- LoadBalancer support in your Kubernetes environment (cloud provider or MetalLB for on-premises)

## Installation

### Quick Start

To install the chart with the release name `my-nginx`:

```bash
helm install my-nginx ./nginx-chart
```

### Custom Installation

To install with custom values:

```bash
helm install my-nginx ./nginx-chart --set replicaCount=3 --set service.type=NodePort
```

### Using values file

1. Create a custom values file:

```yaml
# custom-values.yaml
replicaCount: 4
image:
  tag: "1.25-alpine"
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

2. Install using the custom values:

```bash
helm install my-nginx ./nginx-chart -f custom-values.yaml
```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of nginx replicas | `2` |
| `image.repository` | nginx image repository | `nginx` |
| `image.tag` | nginx image tag | `"1.25"` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `service.type` | Kubernetes service type | `LoadBalancer` |
| `service.port` | Service port | `80` |
| `resources.limits.cpu` | CPU resource limit | `100m` |
| `resources.limits.memory` | Memory resource limit | `128Mi` |
| `resources.requests.cpu` | CPU resource request | `50m` |
| `resources.requests.memory` | Memory resource request | `64Mi` |
| `ingress.enabled` | Enable ingress controller resource | `false` |
| `autoscaling.enabled` | Enable horizontal pod autoscaler | `false` |

For a full list of configurable parameters, see [`values.yaml`](nginx-chart/values.yaml).

## Usage Examples

### Example 1: Basic deployment with LoadBalancer
```bash
helm install my-nginx ./nginx-chart
```

### Example 2: Scale to 5 replicas
```bash
helm install my-nginx ./nginx-chart --set replicaCount=5
```

### Example 3: Use NodePort instead of LoadBalancer
```bash
helm install my-nginx ./nginx-chart --set service.type=NodePort
```

### Example 4: Enable ingress with custom hostname
```bash
helm install my-nginx ./nginx-chart \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=my-nginx.example.com \
  --set ingress.hosts[0].paths[0].path=/ \
  --set ingress.hosts[0].paths[0].pathType=Prefix
```

## Accessing nginx

After installation, follow the instructions in the NOTES output to access your nginx deployment.

For LoadBalancer service (default):
```bash
# Wait for external IP to be assigned
kubectl get svc -w <release-name>-nginx-chart

# Get the external IP
export SERVICE_IP=$(kubectl get svc <release-name>-nginx-chart --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
echo "Application URL: http://$SERVICE_IP:80"
```

## Monitoring and Management

### Check deployment status
```bash
kubectl get all -l "app.kubernetes.io/instance=<release-name>"
```

### View pod logs
```bash
kubectl logs -l "app.kubernetes.io/name=nginx-chart,app.kubernetes.io/instance=<release-name>"
```

### Scaling the deployment
```bash
helm upgrade <release-name> ./nginx-chart --set replicaCount=5
```

## Upgrading

To upgrade your deployment:

```bash
helm upgrade my-nginx ./nginx-chart
```

## Uninstalling

To uninstall the chart:

```bash
helm uninstall my-nginx
```

This will remove all Kubernetes components associated with the chart and delete the release.

## Development

### Testing the chart locally

1. Lint the chart:
```bash
helm lint ./nginx-chart
```

2. Test template rendering:
```bash
helm template test-release ./nginx-chart
```

3. Test installation (dry-run):
```bash
helm install test-release ./nginx-chart --dry-run --debug
```

### Chart Structure

```
nginx-chart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── templates/
│   ├── deployment.yaml # nginx Deployment
│   ├── service.yaml    # LoadBalancer Service
│   ├── serviceaccount.yaml # Service Account
│   ├── ingress.yaml    # Ingress (optional)
│   ├── hpa.yaml        # Horizontal Pod Autoscaler (optional)
│   ├── NOTES.txt       # Post-installation notes
│   └── _helpers.tpl    # Template helpers
└── tests/
    └── test-connection.yaml # Connection test
```

## Troubleshooting

### LoadBalancer IP pending
If your LoadBalancer service shows `<pending>` for the external IP:
- Ensure your Kubernetes cluster supports LoadBalancer services
- For on-premises clusters, consider using MetalLB or similar solutions
- Check with your cloud provider for any quotas or issues

### Pods not starting
Check pod events and logs:
```bash
kubectl describe pods -l "app.kubernetes.io/name=nginx-chart"
kubectl logs -l "app.kubernetes.io/name=nginx-chart"
```

### Resource issues
If pods are pending due to resource constraints:
```bash
kubectl describe nodes
kubectl top pods
```

Consider adjusting resource requests/limits in values.yaml.

## Chart Repository

This chart is automatically published to GitHub Pages at: https://albal.github.io/nginx-helm

### Adding the repository

```bash
helm repo add nginx-helm https://albal.github.io/nginx-helm
helm repo update
```

### Installing from the repository

```bash
helm install my-nginx nginx-helm/nginx-chart
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly (run `helm lint nginx-chart`)
5. Submit a pull request

### Publishing Process

Charts are automatically published when:
- Changes are pushed to the `main` branch
- Version tags (e.g., `v1.0.1`) are created

The GitHub Action workflow will:
1. Lint and test the chart
2. Package the chart
3. Create a GitHub release
4. Update the Helm repository index on GitHub Pages

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

## Support

For support and questions:
- Create an issue in this repository
- Check the troubleshooting section above
- Review Kubernetes and Helm documentation
