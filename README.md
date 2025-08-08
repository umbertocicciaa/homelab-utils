# Homelab Utils

A collection of utilities, configurations, and deployment manifests for my personal homelab running on Raspberry Pi 4 with K3s Kubernetes cluster.

## 🏠 Overview

This repository contains infrastructure-as-code configurations and setup scripts for a self-hosted homelab environment. The homelab is built on Raspberry Pi 4 hardware running a lightweight K3s Kubernetes distribution.

## 🚀 Components

### K3s Installation

- **Location**: `k3s-installation/`
- **Description**: Setup guide and scripts for installing K3s on Raspberry Pi
- **Features**:
  - Master node installation
  - Worker node configuration
  - Memory cgroup configuration for Raspberry Pi OS

### Homepage Dashboard

- **Location**: `homepage/`
- **Description**: Kubernetes manifests for deploying Homepage dashboard
- **Features**:
  - Complete Kubernetes deployment configuration
  - Service definitions and ingress rules
  - RBAC configuration
  - Namespace isolation

## 📋 Prerequisites

- Raspberry Pi 4 (recommended 4GB+ RAM)
- Raspberry Pi OS (64-bit recommended)
- Basic knowledge of Kubernetes and Docker

## 🔧 Installation

### 1. K3s Cluster Setup

Follow the instructions in [`k3s-installation/K3S-INSTALLATION.md`](./k3s-installation/K3S-INSTALLATION.md):

1. Enable memory cgroups on Raspberry Pi OS
2. Install K3s master node
3. Join worker nodes to the cluster

### 2. Deploy Homepage Dashboard

Apply the Kubernetes manifests for the Homepage dashboard:

```bash
# Apply all Homepage configurations
kubectl apply -f homepage/k8s/

# Or apply individually
kubectl apply -f homepage/k8s/namespace.yaml
kubectl apply -f homepage/k8s/configs.yaml
kubectl apply -f homepage/k8s/role.yaml
kubectl apply -f homepage/k8s/deploy.yaml
kubectl apply -f homepage/k8s/ingress.yaml
```

## 📁 Repository Structure

```text
homelab-utils/
├── README.md                    # This file
├── homepage/                    # Homepage dashboard deployment
│   ├── HOMEPAGE.md             # Homepage documentation
│   └── k8s/                    # Kubernetes manifests
│       ├── namespace.yaml      # Namespace definition
│       ├── configs.yaml        # ConfigMaps and Secrets
│       ├── role.yaml          # RBAC configuration
│       ├── deploy.yaml        # Deployment manifest
│       └── ingress.yaml       # Ingress configuration
└── k3s-installation/          # K3s setup guide
    └── K3S-INSTALLATION.md    # Installation instructions
```

## 🛠️ Hardware Setup

- **Platform**: Raspberry Pi 4
- **OS**: Raspberry Pi OS (64-bit)
- **Container Runtime**: containerd (via K3s)
- **Kubernetes Distribution**: K3s (lightweight Kubernetes)

## 📚 Documentation

- [K3s Installation Guide](./k3s-installation/K3S-INSTALLATION.md)
- [Homepage Setup](./homepage/HOMEPAGE.md)

## 🤝 Contributing

This is a personal homelab repository, but feel free to:

- Open issues for questions or suggestions
- Submit pull requests for improvements
- Use these configurations as inspiration for your own homelab

## 📄 License

This project is available under the MIT License. See the LICENSE file for more details.

## 🔗 Useful Links

- [K3s Documentation](https://docs.k3s.io/)
- [Homepage Documentation](https://gethomepage.dev/)
- [Raspberry Pi OS](https://www.raspberrypi.org/software/)

---

**Note**: This homelab setup is designed for learning and personal use. For production environments, consider additional security hardening and high availability configurations.
