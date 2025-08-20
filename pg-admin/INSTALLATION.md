# pgAdmin Installation Guide for K3s

This guide explains how to deploy pgAdmin, a popular web-based PostgreSQL management tool, on your K3s Kubernetes cluster using Helm.

## 📋 Prerequisites

- K3s cluster up and running
- `kubectl` configured to access your cluster
- `helm` installed on your local machine
- (Optional) Persistent storage available for pgAdmin data

## 🚀 Installation

### Step 1: Create Namespace

Create a dedicated namespace for pgAdmin:

```bash
kubectl create namespace pg-admin
```

### Step 2: Add Bitnami Helm Repository

Add and update the Bitnami Helm repository (which provides a maintained pgAdmin chart):

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Step 3: Configure Values

Edit the `pg-admin/values.yaml` file to customize your deployment (admin email, password, persistent storage, ingress, etc.).

Example important values:

```yaml
pgadminUsername: admin@example.com
pgadminPassword: yourStrongPassword
service:
 type: ClusterIP
persistence:
 enabled: true
 size: 1Gi
 storageClass: "local-path"
ingress:
 enabled: true
 hostname: pgadmin.local
```

### Step 4: Install pgAdmin

Install pgAdmin using Helm and your custom values file:

```bash
helm install pgadmin bitnami/pgadmin \
 --namespace pg-admin \
 -f pg-admin/values.yaml
```

### Step 5: Verify Deployment

Check that the pod is running:

```bash
kubectl get pods -n pg-admin
```

Check the service:

```bash
kubectl get svc -n pg-admin
```

## 🔍 Accessing pgAdmin

### Port Forward (Local Access)

```bash
kubectl port-forward svc/pgadmin 8080:80 -n pg-admin
```

Then open [http://localhost:8080](http://localhost:8080) in your browser.

### Ingress (Recommended for Cluster Access)

If you enabled ingress, access pgAdmin at the hostname you configured (e.g., <http://pgadmin.local>). Make sure your DNS or `/etc/hosts` is set up accordingly.

## 🔑 Login Credentials

- **Username:** (as set in `values.yaml`, e.g., `admin@example.com`)
- **Password:** (as set in `values.yaml`)

## ⚙️ Configuration Management

To update your configuration, edit `pg-admin/values.yaml` and run:

```bash
helm upgrade pgadmin bitnami/pgadmin \
 --namespace pg-admin \
 -f pg-admin/values.yaml
```

## 🗑️ Uninstallation

To remove pgAdmin and all its resources:

```bash
helm uninstall pgadmin -n pg-admin
kubectl delete namespace pg-admin
```

## 🔧 Troubleshooting

- Check pod logs:  
 `kubectl logs -n pg-admin <pgadmin-pod-name>`
- Check events:  
 `kubectl get events -n pg-admin`
- Verify persistent volume claims:  
 `kubectl get pvc -n pg-admin`

## 🔗 Useful Links

- [Bitnami pgAdmin Helm Chart](https://artifacthub.io/packages/helm/bitnami/pgadmin)
- [pgAdmin Documentation](https://www.pgadmin.org/docs/)

---
