# K3s Installation Guide for Raspberry Pi

This guide provides step-by-step instructions for installing K3s (lightweight Kubernetes) on Raspberry Pi 4 devices to create a homelab cluster.

## 📋 Prerequisites

### Hardware Requirements

- Raspberry Pi 4 (4GB RAM recommended, 2GB minimum)
- MicroSD card (32GB+ recommended, Class 10)
- Stable network connection
- Power supply (official Raspberry Pi power adapter recommended)

### Software Requirements

- Raspberry Pi OS (64-bit recommended)
- SSH access configured
- Static IP addresses configured (recommended)

## 🔧 Preparation Steps

### 1. Configure Memory Cgroups

K3s requires memory cgroups to be enabled on Raspberry Pi OS. This must be done on **all nodes** (master and workers).

```bash
# Edit the boot configuration file
sudo nano /boot/firmware/cmdline.txt
```

Add the following parameters to the end of the line (ensure it's all on one line):

```text
cgroup_memory=1 cgroup_enable=memory
```

**Example**: If your `cmdline.txt` contains:

```text
console=serial0,115200 console=tty1 root=PARTUUID=12345678-02 rootfstype=ext4 fsck.repair=yes rootwait
```

It should become:

```text
console=serial0,115200 console=tty1 root=PARTUUID=12345678-02 rootfstype=ext4 fsck.repair=yes rootwait cgroup_memory=1 cgroup_enable=memory
```

### 2. Reboot the System

After editing the cmdline.txt file, reboot to apply the changes:

```bash
sudo reboot
```

### 3. Verify Cgroup Configuration

After reboot, verify that cgroups are properly configured:

```bash
# Check if memory cgroup is available
cat /proc/cgroups | grep memory

# Should show something like:
# memory  1  <number>  1
```

## 🚀 K3s Installation

### Master Node Installation

On the designated master node (control plane):

```bash
# Install K3s master node
curl -sfL https://get.k3s.io | sh -

# Wait for the node to be ready
sudo k3s kubectl get node
```

#### Retrieve Node Token

After successful installation, get the node token for worker nodes:

```bash
# Get the node token (needed for worker nodes)
sudo cat /var/lib/rancher/k3s/server/node-token
```

Save this token as you'll need it for worker node installation.

#### Get Master Node IP

```bash
# Get the master node IP address
hostname -I | awk '{print $1}'
```

### Worker Node Installation

On each worker node, run the following command (replace `<MASTER_IP>` and `<NODE_TOKEN>` with actual values):

```bash
# Install K3s worker node
curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER_IP>:6443 K3S_TOKEN=<NODE_TOKEN> sh -
```

**Example**:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.100:6443 K3S_TOKEN=K107c4e5c1b8b4c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4 sh -
```

## ✅ Verification

### Check Cluster Status

On the master node, verify that all nodes have joined the cluster:

```bash
# Check all nodes in the cluster
sudo k3s kubectl get nodes

# Check node details
sudo k3s kubectl get nodes -o wide

# Check system pods
sudo k3s kubectl get pods -A
```

Expected output should show all nodes in `Ready` status.

### Test Cluster Functionality

Deploy a test application to verify the cluster is working:

```bash
# Create a test deployment
sudo k3s kubectl create deployment hello-k3s --image=rancher/hello-world

# Expose the deployment
sudo k3s kubectl expose deployment hello-k3s --type=LoadBalancer --port=80

# Check the service
sudo k3s kubectl get services
```

## 🛠️ Configuration

### Configure kubectl (Optional)

To use `kubectl` without sudo:

```bash
# Copy the kubeconfig file
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config

# Now you can use kubectl without sudo
kubectl get nodes
```

### Enable kubectl Auto-completion

```bash
# Add to your shell profile (~/.bashrc or ~/.zshrc)
echo 'source <(kubectl completion bash)' >> ~/.bashrc  # for bash
echo 'source <(kubectl completion zsh)' >> ~/.zshrc    # for zsh

# Reload your shell
source ~/.bashrc  # or source ~/.zshrc
```

## 🔒 Security Considerations

### Firewall Configuration

Ensure the following ports are open between nodes:

- **6443**: Kubernetes API server
- **10250**: Kubelet API
- **8472**: Flannel VXLAN (if using Flannel CNI)

### Node Hardening

Consider implementing additional security measures:

1. Change default SSH port
2. Disable password authentication (use SSH keys)
3. Configure fail2ban
4. Keep the system updated

## 🚨 Troubleshooting

### Common Issues

1. **Nodes not joining cluster**:
   - Verify network connectivity between nodes
   - Check firewall settings
   - Ensure correct master IP and token

2. **Memory issues**:
   - Verify cgroup configuration
   - Check available memory: `free -h`

3. **Pod scheduling issues**:
   - Check node resources: `kubectl describe nodes`
   - Verify node taints: `kubectl get nodes -o yaml`

### Useful Commands

```bash
# Check K3s service status
sudo systemctl status k3s

# View K3s logs
sudo journalctl -u k3s -f

# Restart K3s service
sudo systemctl restart k3s

# Check cluster info
kubectl cluster-info

# Get all resources
kubectl get all -A
```

## 🗑️ Uninstallation

### Remove K3s from Master Node

```bash
sudo /usr/local/bin/k3s-uninstall.sh
```

### Remove K3s from Worker Nodes

```bash
sudo /usr/local/bin/k3s-agent-uninstall.sh
```

## 📚 Additional Resources

- [K3s Official Documentation](https://docs.k3s.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/)

## 📝 Notes

- K3s uses SQLite as the default datastore for single-node clusters
- For high availability, consider using etcd or external databases
- Default CNI is Flannel, but can be replaced with others like Cilium
- K3s includes Traefik as the default ingress controller
- Local storage provisioner is included by default

---

**Last Updated**: August 2025
