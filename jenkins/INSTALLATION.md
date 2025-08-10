# Jenkins Installation Guide for K3s

This guide provides step-by-step instructions for installing and configuring Jenkins on your K3s Kubernetes cluster using Helm with persistent storage and proper RBAC configuration.

## 📋 Overview

Jenkins is an open-source automation server that enables developers to build, test, and deploy their applications reliably. This installation includes:

- **Jenkins Controller**: Main Jenkins instance with web UI
- **Persistent Storage**: Data persistence using hostPath volumes
- **RBAC Configuration**: Service account with cluster permissions
- **Ingress Access**: External access via Traefik ingress controller
- **Kubernetes Integration**: Native Kubernetes plugin support

## 📋 Prerequisites

- K3s cluster up and running
- `kubectl` configured to access your cluster
- `helm` installed on your local machine
- At least 4GB of available memory on your cluster nodes
- 20GB+ available disk space for Jenkins data

## 🚀 Installation

### Step 1: Create Jenkins Namespace

Create a dedicated namespace for Jenkins:

```bash
kubectl create namespace jenkins
```

### Step 2: Set Up Persistent Storage

Create the persistent volume and storage class for Jenkins data:

```bash
# Apply the persistent volume configuration
kubectl apply -f jenkins/jenkins-01-volume.yaml
```

**Note**: The persistent volume uses a hostPath at `/data/jenkins-volume/`. Make sure this directory exists on your K3s node:

```bash
# On your K3s node
sudo mkdir -p /data/jenkins-volume/
sudo chown -R 1000:1000 /data/jenkins-volume/
```

### Step 3: Configure RBAC

Apply the service account and cluster role for Jenkins:

```bash
# Apply RBAC configuration
kubectl apply -f jenkins/jenkins-02-sa.yaml
```

This creates:

- Service account `jenkins` in the `jenkins` namespace
- ClusterRole with permissions to manage Kubernetes resources
- ClusterRoleBinding to associate the service account with the role

### Step 4: Add Jenkins Helm Repository

Add and update the Jenkins Helm repository:

```bash
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
```

Verify the available charts:

```bash
helm search repo jenkinsci
```

### Step 5: Install Jenkins with Helm

Install Jenkins using the custom values file:

```bash
helm install jenkins jenkinsci/jenkins \
  --namespace jenkins \
  --values jenkins/jenkins-values.yaml
```

### Step 6: Verify Installation

Check that all pods are running successfully:

```bash
kubectl --namespace jenkins get pods
```

You should see:

- `jenkins-0` (StatefulSet pod)

Check the services:

```bash
kubectl --namespace jenkins get svc
```

## 🔍 Accessing Jenkins

### Get Admin Password

Retrieve the Jenkins admin password:

```bash
kubectl --namespace jenkins get secret jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 -d ; echo
```

### Access via Port Forward

Forward Jenkins port to your local machine:

```bash
kubectl --namespace jenkins port-forward svc/jenkins 8080:8080
```

Then navigate to: <http://localhost:8080>

### Access via Ingress (if configured)

If you've configured ingress in the values file, Jenkins will be accessible at:

- <http://your-cluster-ip/jenkins>

**Login Credentials:**

- Username: `admin`
- Password: Use the password retrieved above

## ⚙️ Configuration Management

### Updating Jenkins Configuration

To modify Jenkins settings:

1. Edit the `jenkins/jenkins-values.yaml` file
2. Update the deployment using Helm upgrade:

```bash
helm upgrade jenkins jenkinsci/jenkins \
  --namespace jenkins \
  --values jenkins/jenkins-values.yaml
```

### Key Configuration Options

The `jenkins-values.yaml` file includes:

- **Controller Image**: Jenkins LTS version
- **Admin Credentials**: Default admin user configuration
- **Persistent Storage**: Volume mounts and storage class
- **Service Configuration**: ClusterIP service type
- **Ingress Settings**: Traefik ingress with path-based routing
- **Resource Limits**: CPU and memory constraints
- **RBAC**: Service account integration

### Installing Plugins

Jenkins comes with essential plugins pre-installed. To add more plugins:

1. Access Jenkins web UI
2. Navigate to "Manage Jenkins" > "Manage Plugins"
3. Install required plugins from the "Available" tab

Recommended plugins for Kubernetes:

- Kubernetes Plugin
- Docker Pipeline
- Blue Ocean
- Pipeline Stage View

## 🔧 Kubernetes Integration

### Configure Kubernetes Cloud

1. Go to "Manage Jenkins" > "Manage Nodes and Clouds" > "Configure Clouds"
2. Add a new "Kubernetes" cloud
3. Configure the following:
   - **Kubernetes URL**: `https://kubernetes.default`
   - **Kubernetes Namespace**: `jenkins`
   - **Credentials**: Use the service account token
   - **Jenkins URL**: `http://jenkins.jenkins.svc.cluster.local:8080`

### Pod Templates

Create pod templates for running Jenkins agents:

```yaml
# Example pod template for Maven builds
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8.1-adoptopenjdk-11
    command:
    - cat
    tty: true
  - name: docker
    image: docker:dind
    securityContext:
      privileged: true
```

## 🗑️ Uninstallation

### Remove Helm Release

```bash
helm uninstall jenkins -n jenkins
```

### Clean Up Persistent Data (Optional)

**Warning**: This will delete all Jenkins data permanently.

```bash
# Remove persistent volume claims
kubectl -n jenkins delete pvc --all

# Remove persistent volume
kubectl delete pv jenkins-pv

# Remove data directory (on K3s node)
sudo rm -rf /data/jenkins-volume/
```

### Remove RBAC and Storage Resources

```bash
kubectl delete -f jenkins/jenkins-02-sa.yaml
kubectl delete -f jenkins/jenkins-01-volume.yaml
```

### Remove Namespace

```bash
kubectl delete namespace jenkins
```

## 🔧 Troubleshooting

### Common Issues

#### Pod Stuck in Pending State

Check resource availability and storage:

```bash
kubectl describe pod jenkins-0 -n jenkins
kubectl get pv
kubectl get pvc -n jenkins
```

#### Permission Issues

Verify the data directory permissions on the K3s node:

```bash
ls -la /data/jenkins-volume/
sudo chown -R 1000:1000 /data/jenkins-volume/
```

#### Ingress Not Working

Check ingress configuration and Traefik:

```bash
kubectl get ingress -n jenkins
kubectl get svc -n kube-system
```

### Logs Inspection

View Jenkins logs:

```bash
# Controller logs
kubectl -n jenkins logs jenkins-0

# Recent logs with follow
kubectl -n jenkins logs -f jenkins-0
```

### Resource Monitoring

Monitor resource usage:

```bash
kubectl top pod jenkins-0 -n jenkins
kubectl top node
```

## 📊 Jenkins Configuration as Code (JCasC)

For advanced configuration, consider using Jenkins Configuration as Code:

1. Create a ConfigMap with JCasC YAML
2. Mount it in the Jenkins pod
3. Set the `CASC_JENKINS_CONFIG` environment variable

Example JCasC configuration:

```yaml
jenkins:
  systemMessage: "Welcome to our K3s Jenkins Instance"
  numExecutors: 0
  scmCheckoutRetryCount: 2
  
security:
  globalJobDslSecurityConfiguration:
    useScriptSecurity: false
```

## 🔗 Useful Links

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Jenkins Helm Chart](https://github.com/jenkinsci/helm-charts)
- [Kubernetes Plugin Documentation](https://plugins.jenkins.io/kubernetes/)
- [Jenkins Configuration as Code](https://jenkins.io/projects/jcasc/)

---

**Note**: This setup is designed for homelab and development environments. For production use, consider additional security hardening, backup strategies, and high availability configurations.
