# Homepage Dashboard Deployment

This guide provides instructions for deploying Homepage, a modern, fully static, fast dashboard for your homelab, on a K3s Kubernetes cluster.

## 📋 Overview

Homepage is a customizable dashboard that provides a central view of your homelab services, applications, and system resources. This deployment is configured to run on Kubernetes with:

- Kubernetes cluster monitoring
- Resource usage widgets
- Service bookmarks and links
- Dynamic service discovery
- Custom styling support

## 🏗️ Architecture

The deployment consists of the following Kubernetes components:

- **Namespace**: `homepage` - Isolated environment for the application
- **Deployment**: Homepage application container with configuration mounts
- **Service**: Internal cluster service for pod access
- **Ingress**: External access routing
- **ConfigMap**: Application configuration and settings
- **RBAC**: Service account with cluster read permissions for monitoring

## 📋 Prerequisites

- K3s cluster up and running
- `kubectl` configured to access your cluster
- Ingress controller (Traefik is included with K3s by default)

## 🚀 Quick Start

### Deploy Homepage

Apply all the Kubernetes manifests:

```bash
# Navigate to the k8s directory
cd homepage/k8s/

# Apply all manifests
kubectl apply -f .

# Or apply individually in order
kubectl apply -f namespace.yaml
kubectl apply -f configs.yaml
kubectl apply -f role.yaml
kubectl apply -f deploy.yaml
kubectl apply -f ingress.yaml
```

### Verify Deployment

Check that all components are running:

```bash
# Check namespace
kubectl get namespace homepage

# Check all resources in the homepage namespace
kubectl get all -n homepage

# Check pod logs
kubectl logs -n homepage deployment/homepage

# Check ingress
kubectl get ingress -n homepage
```

## 🔧 Configuration

### Core Configuration Files

The Homepage configuration is managed through a ConfigMap with the following files:

#### `settings.yaml`

Global application settings (currently empty in this deployment):

```yaml
settings.yaml: ""
```

#### `services.yaml`

Define your homelab services and applications:

```yaml
services.yaml: |
  - My First Group:
      - My First Service:
          href: http://localhost/
          description: Homepage is awesome

  - My Second Group:
      - My Second Service:
          href: http://localhost/
          description: Homepage is the best

  - My Third Group:
      - My Third Service:
          href: http://localhost/
          description: Homepage is 😎
```

#### `widgets.yaml`

Configure dashboard widgets for monitoring:

```yaml
widgets.yaml: |
  - kubernetes:
      cluster:
        show: true
        cpu: true
        memory: true
        showLabel: true
        label: "cluster"
      nodes:
        show: true
        cpu: true
        memory: true
        showLabel: true
  - resources:
      backend: resources
      expanded: true
      cpu: true
      memory: true
      network: default
  - search:
      provider: duckduckgo
      target: _blank
```

#### `bookmarks.yaml`

Quick access bookmarks:

```yaml
bookmarks.yaml: |
  - Developer:
      - Github:
          - abbr: GH
            href: https://github.com/
```

#### `kubernetes.yaml`

Kubernetes integration settings:

```yaml
kubernetes.yaml: |
  mode: cluster
```

### Customization Files

- `custom.css`: Custom CSS styling (empty by default)
- `custom.js`: Custom JavaScript (empty by default)
- `docker.yaml`: Docker integration (empty in this deployment)

## 🔐 Security Configuration

### RBAC Permissions

The deployment includes a service account with the following cluster permissions:

- **Read access** to:
  - Namespaces
  - Pods
  - Nodes
  - Ingresses (networking.k8s.io, extensions)
  - IngressRoutes (traefik.io)
  - HTTPRoutes and Gateways (gateway.networking.k8s.io)
  - Metrics (metrics.k8s.io)

### Environment Variables

- `HOMEPAGE_ALLOWED_HOSTS`: Set to `gethomepage.dev` (configure as needed)

## 🌐 Access

### Default Access

By default, Homepage is accessible through:

- **Internal**: `http://homepage.homepage.svc.cluster.local:3000`
- **Ingress**: Root path `/` (configure your domain as needed)

### Configuring Domain Access

To access Homepage via a custom domain:

1. Update the ingress configuration in `ingress.yaml`:

```yaml
spec:
  rules:
    - host: homepage.yourdomain.com
      http:
        paths:
          - path: "/"
            pathType: Prefix
            backend:
              service:
                name: homepage
                port:
                  number: 3000
```

1. Configure DNS to point to your cluster's external IP

## 🎨 Customization

### Adding Services

Edit the ConfigMap to add your homelab services:

```bash
# Edit the configmap
kubectl edit configmap homepage -n homepage

# Or update the configs.yaml file and reapply
kubectl apply -f configs.yaml
```

### Service Configuration Example

```yaml
services.yaml: |
  - Infrastructure:
      - Proxmox:
          href: https://proxmox.local:8006
          description: Virtualization Management
          icon: proxmox.png
      - pfSense:
          href: https://pfsense.local
          description: Firewall & Router
          icon: pfsense.png
  
  - Media:
      - Plex:
          href: https://plex.local:32400
          description: Media Server
          icon: plex.png
      - Jellyfin:
          href: https://jellyfin.local:8096
          description: Alternative Media Server
          icon: jellyfin.png
  
  - Monitoring:
      - Grafana:
          href: https://grafana.local:3000
          description: Metrics Dashboard
          icon: grafana.png
      - Prometheus:
          href: https://prometheus.local:9090
          description: Metrics Collection
          icon: prometheus.png
```

### Widget Configuration

Add more widgets for system monitoring:

```yaml
widgets.yaml: |
  - kubernetes:
      cluster:
        show: true
        cpu: true
        memory: true
        showLabel: true
        label: "cluster"
      nodes:
        show: true
        cpu: true
        memory: true
        showLabel: true
  
  - resources:
      backend: resources
      expanded: true
      cpu: true
      memory: true
      network: default
  
  - search:
      provider: duckduckgo
      target: _blank
  
  - openmeteo:
      label: Weather
      latitude: 40.7128
      longitude: -74.0060
      timezone: America/New_York
      units: metric
      cache: 5
```

## 🔧 Resource Configuration

### Container Resources

The deployment is configured with:

- **Requests**: 100m CPU, 128Mi memory
- **Limits**: 500m CPU, 512Mi memory

Adjust these based on your cluster resources:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

### Volume Mounts

All configuration files are mounted from the ConfigMap to `/app/config/`

## 📊 Monitoring

### Health Checks

Homepage runs on port 3000 and provides health endpoints:

```bash
# Port forward to access locally
kubectl port-forward -n homepage deployment/homepage 3000:3000

# Access Homepage
open http://localhost:3000
```

### Troubleshooting

```bash
# Check pod status
kubectl get pods -n homepage

# View logs
kubectl logs -n homepage deployment/homepage

# Describe deployment for events
kubectl describe deployment homepage -n homepage

# Check configmap
kubectl get configmap homepage -n homepage -o yaml

# Check service account permissions
kubectl auth can-i get pods --as=system:serviceaccount:homepage:homepage
```

## 🗑️ Uninstallation

To remove Homepage from your cluster:

```bash
# Delete all resources
kubectl delete -f homepage/k8s/

# Or delete the namespace (removes everything)
kubectl delete namespace homepage
```

## 📚 Additional Resources

- [Homepage Official Documentation](https://gethomepage.dev/)
- [Homepage GitHub Repository](https://github.com/gethomepage/homepage)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

## 📝 Notes

- Homepage automatically discovers services with proper annotations
- The deployment uses the latest image tag for automatic updates
- ConfigMap changes require pod restart to take effect
- The service account has cluster-wide read permissions for monitoring

## 🔄 Updates

To update Homepage:

```bash
# Update the deployment to pull latest image
kubectl rollout restart deployment homepage -n homepage

# Check rollout status
kubectl rollout status deployment homepage -n homepage
```

---

**Last Updated**: August 2025