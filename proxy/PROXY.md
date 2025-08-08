# Local Machine Proxy Setup for Raspberry Pi Cluster

A comprehensive guide for configuring your local machine as a proxy to access your Raspberry Pi K3s cluster remotely, including kubectl access and SSH tunneling.

## 📋 Overview

This guide covers how to use your local machine as a gateway to access your homelab Raspberry Pi cluster when you're not on the same local network. This setup enables:

- **Remote kubectl access** to your K3s cluster
- **SSH tunneling** through your local machine to cluster nodes
- **Port forwarding** for accessing cluster services
- **Secure remote access** to your homelab infrastructure

## 🏗️ Architecture

```text
Internet → Local Machine (Proxy) → Home Network → Raspberry Pi Cluster
                ↓
        - SSH Tunnels
        - Kubectl Access
        - Port Forwarding
        - VPN Gateway
```

## 📋 Prerequisites

### Local Machine Requirements

- **Static public IP** or **Dynamic DNS** service
- **SSH server** running and accessible from internet
- **kubectl** installed
- **SSH client** with key authentication
- **Firewall** properly configured

### Raspberry Pi Cluster Requirements

- **K3s cluster** up and running
- **SSH access** configured on all nodes
- **Network connectivity** between local machine and cluster
- **Kubeconfig file** available

## 🚀 Setup Process

### Step 1: Configure Local Machine SSH Server

#### Enable SSH Server (if not already running)

**On macOS:**

```bash
# Enable Remote Login in System Preferences > Sharing
# Or via command line:
sudo systemsetup -setremotelogin on

# Check SSH service status
sudo launchctl list | grep ssh
```

**On Linux:**

```bash
# Install and enable SSH server
sudo apt update
sudo apt install -y openssh-server

# Enable and start SSH service
sudo systemctl enable ssh
sudo systemctl start ssh

# Check status
sudo systemctl status ssh
```

#### Configure SSH Security

```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
```

**Recommended security settings:**

```bash
# Change default port (recommended)
Port 2222

# Disable root login
PermitRootLogin no

# Enable key authentication
PubkeyAuthentication yes

# Disable password authentication (after setting up keys)
PasswordAuthentication no

# Limit login attempts
MaxAuthTries 3

# Set allowed users (replace 'yourusername')
AllowUsers yourusername

# Enable TCP forwarding for tunnels
AllowTcpForwarding yes
GatewayPorts no
```

#### Restart SSH Service

```bash
# On macOS
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load /System/Library/LaunchDaemons/ssh.plist

# On Linux
sudo systemctl restart ssh
```

### Step 2: Configure Firewall and Router

#### Local Machine Firewall

**On macOS:**

```bash
# Allow SSH through firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /usr/sbin/sshd
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /usr/sbin/sshd
```

**On Linux:**

```bash
# Configure UFW firewall
sudo ufw allow 2222/tcp
sudo ufw enable
sudo ufw status
```

#### Router Port Forwarding

Configure your router to forward SSH traffic:

- **External Port**: 2222 (or your chosen port)
- **Internal Port**: 2222
- **Protocol**: TCP
- **Destination**: Your local machine's IP

### Step 3: Set Up SSH Key Authentication

#### Generate SSH Key Pair (if needed)

```bash
# Generate a new SSH key pair
ssh-keygen -t rsa -b 4096 -C "homelab-proxy-access"

# Save to a specific location
# ~/.ssh/homelab_proxy_rsa (recommended)
```

#### Copy Public Key to Raspberry Pi Nodes

```bash
# Copy key to each cluster node
ssh-copy-id -i ~/.ssh/homelab_proxy_rsa.pub pi@192.168.1.100
ssh-copy-id -i ~/.ssh/homelab_proxy_rsa.pub pi@192.168.1.101
ssh-copy-id -i ~/.ssh/homelab_proxy_rsa.pub pi@192.168.1.102
```

### Step 4: Configure Kubectl Access

#### Copy Kubeconfig from Cluster

```bash
# SSH to master node and copy kubeconfig
ssh pi@192.168.1.100 "sudo cat /etc/rancher/k3s/k3s.yaml" > ~/.kube/homelab-config

# Edit the server URL to use your home IP
nano ~/.kube/homelab-config
```

**Update server URL in kubeconfig:**

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0t...
    server: https://YOUR_HOME_PUBLIC_IP:6443  # Change this
  name: default
```

#### Set Up kubectl Context

```bash
# Set KUBECONFIG environment variable
export KUBECONFIG=~/.kube/homelab-config

# Or merge with existing config
kubectl config --kubeconfig=~/.kube/homelab-config view --flatten > ~/.kube/config-merged
cp ~/.kube/config-merged ~/.kube/config

# Test connection
kubectl get nodes
```

## 🔧 Access Methods

### Method 1: Direct SSH Tunneling

#### SSH to Cluster Nodes via Local Machine

```bash
# SSH to cluster through your local machine
ssh -J yourusername@YOUR_PUBLIC_IP:2222 pi@192.168.1.100

# Or with specific key
ssh -J yourusername@YOUR_PUBLIC_IP:2222 -i ~/.ssh/homelab_proxy_rsa pi@192.168.1.100
```

#### Create SSH Config for Easy Access

```bash
# Edit SSH config
nano ~/.ssh/config
```

**Add configuration:**

```bash
# Local machine (proxy)
Host homelab-proxy
    HostName YOUR_PUBLIC_IP
    Port 2222
    User yourusername
    IdentityFile ~/.ssh/homelab_proxy_rsa

# Cluster master node
Host rpi-master
    HostName 192.168.1.100
    User pi
    ProxyJump homelab-proxy
    IdentityFile ~/.ssh/homelab_proxy_rsa

# Cluster worker nodes
Host rpi-worker1
    HostName 192.168.1.101
    User pi
    ProxyJump homelab-proxy
    IdentityFile ~/.ssh/homelab_proxy_rsa

Host rpi-worker2
    HostName 192.168.1.102
    User pi
    ProxyJump homelab-proxy
    IdentityFile ~/.ssh/homelab_proxy_rsa
```

**Usage:**

```bash
# Connect to nodes easily
ssh rpi-master
ssh rpi-worker1
ssh rpi-worker2
```

### Method 2: Kubectl Port Forwarding

#### Access Kubernetes API

```bash
# Create tunnel to Kubernetes API server
ssh -L 6443:192.168.1.100:6443 yourusername@YOUR_PUBLIC_IP -p 2222

# In another terminal, use kubectl with localhost
export KUBECONFIG=~/.kube/homelab-config-local
# Update server URL to https://localhost:6443
kubectl get nodes
```

#### Access Cluster Services

```bash
# Forward service ports (example: Homepage dashboard)
kubectl port-forward -n homepage svc/homepage 8080:3000

# Access via http://localhost:8080
```

### Method 3: VPN-like Access with SOCKS Proxy

#### Create SOCKS Proxy

```bash
# Create SOCKS proxy tunnel
ssh -D 8080 -N yourusername@YOUR_PUBLIC_IP -p 2222
```

#### Configure Browser for SOCKS Proxy

Configure your browser to use `localhost:8080` as SOCKS proxy, then access cluster services directly via their internal IPs.

### Method 4: Persistent SSH Tunnels

#### Create Persistent Tunnels with autossh

```bash
# Install autossh
brew install autossh  # macOS
sudo apt install autossh  # Linux

# Create persistent tunnel
autossh -M 20000 -L 6443:192.168.1.100:6443 yourusername@YOUR_PUBLIC_IP -p 2222 -N
```

#### Create Systemd Service (Linux)

```bash
# Create service file
sudo nano /etc/systemd/system/homelab-tunnel.service
```

**Service configuration:**

```ini
[Unit]
Description=SSH tunnel to homelab cluster
After=network.target

[Service]
Type=simple
User=yourusername
ExecStart=/usr/bin/autossh -M 20000 -N -L 6443:192.168.1.100:6443 yourusername@YOUR_PUBLIC_IP -p 2222
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start service
sudo systemctl enable homelab-tunnel
sudo systemctl start homelab-tunnel
sudo systemctl status homelab-tunnel
```

## 🛠️ Advanced Configuration

### Dynamic Port Forwarding Script

Create a script for easy port forwarding:

```bash
#!/bin/bash
# ~/bin/homelab-forward.sh

PROXY_HOST="yourusername@YOUR_PUBLIC_IP"
PROXY_PORT="2222"

# Function to forward a port
forward_port() {
    local local_port=$1
    local remote_host=$2
    local remote_port=$3
    local service_name=$4
    
    echo "Forwarding $service_name: localhost:$local_port → $remote_host:$remote_port"
    ssh -L $local_port:$remote_host:$remote_port $PROXY_HOST -p $PROXY_PORT -N &
    echo $! > /tmp/tunnel-$local_port.pid
}

# Kubernetes API
forward_port 6443 192.168.1.100 6443 "Kubernetes API"

# Homepage Dashboard
forward_port 8080 192.168.1.100 80 "Homepage Dashboard"

# Add more services as needed
echo "Tunnels established. Press Ctrl+C to stop all tunnels."

# Cleanup function
cleanup() {
    echo "Stopping all tunnels..."
    for pidfile in /tmp/tunnel-*.pid; do
        if [ -f "$pidfile" ]; then
            kill $(cat "$pidfile") 2>/dev/null
            rm "$pidfile"
        fi
    done
    exit 0
}

trap cleanup INT
wait
```

Make it executable:

```bash
chmod +x ~/bin/homelab-forward.sh
```

### kubectl Context Switching

Create multiple kubectl contexts for different access methods:

```bash
# Direct access (when on local network)
kubectl config set-context homelab-direct \
    --cluster=default \
    --user=default \
    --namespace=default

# Tunneled access (when remote)
kubectl config set-context homelab-tunnel \
    --cluster=default \
    --user=default \
    --namespace=default

# Switch contexts
kubectl config use-context homelab-tunnel
kubectl config use-context homelab-direct
```

## 🔐 Security Considerations

### SSH Security Best Practices

1. **Use strong SSH keys** (RSA 4096-bit or Ed25519)
1. **Disable password authentication**
1. **Change default SSH port**
1. **Implement fail2ban** for intrusion prevention
1. **Regular security updates**

### Network Security

```bash
# Monitor SSH connections
sudo tail -f /var/log/auth.log

# Check active connections
ss -tuln | grep :2222

# Monitor failed login attempts
sudo journalctl -u ssh -f
```

### Firewall Configuration

```bash
# Only allow necessary ports
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp
sudo ufw enable
```

## 📊 Monitoring and Troubleshooting

### Connection Testing

```bash
# Test SSH connectivity
ssh -T yourusername@YOUR_PUBLIC_IP -p 2222

# Test cluster connectivity through tunnel
kubectl --kubeconfig ~/.kube/homelab-config get nodes

# Test port forwarding
curl -k https://localhost:6443/version
```

### Common Issues and Solutions

#### Issue: Connection Refused

```bash
# Check if SSH service is running
sudo systemctl status ssh

# Check firewall rules
sudo ufw status

# Check router port forwarding
```

#### Issue: Kubectl Connection Failed

```bash
# Verify kubeconfig server URL
kubectl config view --minify

# Check if API server tunnel is active
ss -tuln | grep 6443

# Test direct connection to cluster
ping 192.168.1.100
```

#### Issue: SSH Key Permission Denied

```bash
# Fix SSH key permissions
chmod 600 ~/.ssh/homelab_proxy_rsa
chmod 644 ~/.ssh/homelab_proxy_rsa.pub

# Verify key is added to SSH agent
ssh-add ~/.ssh/homelab_proxy_rsa
ssh-add -l
```

### Performance Monitoring

```bash
# Monitor tunnel bandwidth
sudo netstat -i

# Check connection latency
ping YOUR_PUBLIC_IP

# Monitor SSH processes
ps aux | grep ssh
```

## 🚀 Usage Examples

### Daily Workflows

#### Morning Cluster Check

```bash
# Start tunnels
~/bin/homelab-forward.sh &

# Check cluster status
kubectl get nodes
kubectl get pods -A

# Access Homepage dashboard
open http://localhost:8080
```

#### Deploy New Service

```bash
# SSH to master node
ssh rpi-master

# Deploy application
kubectl apply -f deployment.yaml

# Check deployment
kubectl get deployments
```

#### Troubleshooting

```bash
# Access worker node for debugging
ssh rpi-worker1

# Check logs
sudo journalctl -u k3s-agent -f

# Check system resources
htop
```

## 📋 Maintenance Checklist

### Weekly Tasks

- [ ] Test SSH connectivity to all nodes
- [ ] Verify kubectl access works
- [ ] Check tunnel processes are running
- [ ] Monitor SSH logs for suspicious activity
- [ ] Update SSH keys if needed

### Monthly Tasks

- [ ] Update SSH server configuration
- [ ] Review and rotate SSH keys
- [ ] Check firewall rules
- [ ] Test disaster recovery procedures
- [ ] Document any configuration changes

## 📚 Additional Resources

- [SSH Tunneling Guide](https://www.ssh.com/academy/ssh/tunneling)
- [kubectl Proxy Documentation](https://kubernetes.io/docs/tasks/extend-kubernetes/http-proxy-access-api/)
- [autossh Documentation](https://linux.die.net/man/1/autossh)

## 📝 Notes

- Always use SSH keys instead of passwords
- Monitor your public IP for security breaches
- Consider setting up a VPN for additional security
- Keep SSH client and server updated
- Document your tunnel configurations
- Test backup access methods regularly

## 🔗 Integration with Other Components

This proxy setup works seamlessly with:

- [Raspberry Pi Setup](../raspberry-installation/RASPBERRY.md)
- [K3s Installation](../k3s-installation/K3S-INSTALLATION.md)
- [Homepage Dashboard](../homepage/HOMEPAGE.md)

---

**Last Updated**: August 2025