# Observability Stack Installation Guide

This guide provides step-by-step instructions for installing and configuring the complete observability stack (Prometheus + Grafana) on your K3s Kubernetes cluster using the kube-prometheus-stack Helm chart.

## 📋 Overview

The observability stack includes:

- **Prometheus**: Metrics collection and time-series database
- **Grafana**: Visualization and dashboarding
- **AlertManager**: Alert handling and routing
- **Node Exporter**: System metrics collection
- **Kube State Metrics**: Kubernetes object metrics
- **Prometheus Operator**: Kubernetes-native Prometheus management

## 📋 Prerequisites

- K3s cluster up and running
- `kubectl` configured to access your cluster
- `helm` installed on your local machine
- At least 2GB of available memory on your cluster nodes

## 🚀 Installation

### Step 1: Create Monitoring Namespace

Create a dedicated namespace for monitoring components:

```bash
kubectl create namespace monitoring
```

### Step 2: Add Prometheus Community Helm Repository

Add and update the Prometheus Community Helm repository:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Step 3: Install kube-prometheus-stack

Install the complete monitoring stack using the custom values file:

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring -f observability/values.yaml
```

### Step 4: Verify Installation

Check that all pods are running successfully:

```bash
kubectl --namespace monitoring get pods -l "release=kube-prometheus-stack"
```

You should see pods for:

- `prometheus-kube-prometheus-stack-prometheus-*`
- `kube-prometheus-stack-grafana-*`
- `kube-prometheus-stack-operator-*`
- `kube-prometheus-stack-kube-state-metrics-*`

## 🔍 Accessing the Services

### Prometheus UI

Access Prometheus web interface locally:

```bash
kubectl -n monitoring port-forward svc/kube-prometheus-stack-prometheus 9090:9090
```

Then navigate to: <http://localhost:9090>

### Grafana Dashboard

#### Get Admin Password

Retrieve the auto-generated Grafana admin password:

```bash
kubectl --namespace monitoring get secrets kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
```

#### Access Grafana

Forward Grafana port to your local machine:

```bash
export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=kube-prometheus-stack" -oname)
kubectl --namespace monitoring port-forward $POD_NAME 3000
```

Then navigate to: <http://localhost:3000>

**Login Credentials:**

- Username: `admin`
- Password: Use the password retrieved in the previous step

## ⚙️ Configuration Management

### Updating Configuration

To modify Grafana password or other settings:

1. Edit the `observability/values.yaml` file
2. Update the configuration using Helm upgrade:

```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring -f observability/values.yaml
```

### Key Configuration Options

The `values.yaml` file includes:

- **Grafana Admin Password**: Change from default `password`
- **Storage Configuration**: Persistent volumes for data retention
- **Ingress Settings**: External access configuration
- **Resource Limits**: CPU and memory constraints
- **Retention Policies**: Data retention periods

## 🗑️ Uninstallation

### Remove Helm Release

```bash
helm uninstall kube-prometheus-stack -n monitoring
```

### Clean Up Persistent Volumes (Optional)

**Warning**: This will delete all monitoring data permanently.

```bash
# List persistent volume claims
kubectl -n monitoring get pvc

# Delete PVCs to reclaim storage space
kubectl -n monitoring delete pvc --all
```

### Remove Namespace

```bash
kubectl delete namespace monitoring
```

## 🔧 Troubleshooting

### Common Issues

#### Pods Stuck in Pending State

Check resource availability:

```bash
kubectl describe pods -n monitoring
kubectl top nodes
```

#### Storage Issues

Verify storage class availability:

```bash
kubectl get storageclass
```

#### Network Connectivity

Test service connectivity:

```bash
kubectl -n monitoring get svc
kubectl -n monitoring describe svc kube-prometheus-stack-grafana
```

### Logs Inspection

View component logs:

```bash
# Prometheus logs
kubectl -n monitoring logs -l app.kubernetes.io/name=prometheus

# Grafana logs
kubectl -n monitoring logs -l app.kubernetes.io/name=grafana

# Operator logs
kubectl -n monitoring logs -l app.kubernetes.io/name=kube-prometheus-stack
```

## 📊 Default Dashboards

Grafana comes pre-configured with dashboards for:

- **Kubernetes Cluster Overview**
- **Node Exporter Metrics**
- **Pod Resource Usage**
- **Persistent Volume Usage**
- **Network Statistics**
- **Application Performance**

## 🔗 Useful Links

- [kube-prometheus-stack Documentation](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Prometheus Operator](https://prometheus-operator.dev/)