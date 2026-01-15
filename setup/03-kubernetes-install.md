# 03 — Kubernetes Component Installation (kubeadm, kubelet, kubectl)

This document explains how to install the core Kubernetes components required to create and manage a cluster using kubeadm. The steps are aligned with CKA exam expectations and assume a single-node Kubernetes cluster on WSL2.

## Kubernetes Components Overview

Kubernetes is composed of several binaries. For this lab, we install:

- **kubeadm**  
  Used to bootstrap and manage the Kubernetes cluster lifecycle.

- **kubelet**  
  Runs on the node and is responsible for managing Pods and containers.

- **kubectl**  
  Command-line tool used to interact with the Kubernetes API server.

## Prerequisites

Before proceeding, ensure:

- WSL2 is properly configured
- systemd is enabled
- containerd is installed and running
- Swap is disabled

(Completed in `01-wsl-setup.md` and `02-container-runtime.md`.)

## Step 1 — Install Required Packages

Update package index and install prerequisites:

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl
```

## Step 2 — Add Kubernetes Package Repository

Kubernetes binaries are installed from the official Kubernetes package repository.

- **Create keyring directory**
  ```bash
  sudo mkdir -p /etc/apt/keyrings
  ```

- **Add repository signing key**
  ```bash
  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
    | sudo tee /etc/apt/keyrings/kubernetes.asc
  ```

- **Add Kubernetes repository**
  ```bash
  echo "deb [signed-by=/etc/apt/keyrings/kubernetes.asc] \
  https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /" \
    | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```

Update package index:

```bash
sudo apt update
```

## Step 3 — Install Kubernetes Components

Install kubeadm, kubelet, and kubectl:

```bash
sudo apt install -y kubeadm kubelet kubectl
```

## Step 4 — Prevent Automatic Upgrades (Important)

To avoid version drift during CKA preparation, hold the packages:

```bash
sudo apt-mark hold kubeadm kubelet kubectl
```

Verify:

```bash
apt-mark showhold
```

You should see:

```
kubeadm
kubelet
kubectl
```

## Step 5 — Verify Installation

Verify that all components are installed correctly.

```bash
kubeadm version
kubelet --version
kubectl version --client
```

All versions should be v1.34.x.

## Important Notes (CKA Context)

- You are not required to install Kubernetes in the CKA exam
- However, understanding these components helps with troubleshooting
- Version consistency is important during preparation

## Result

At this point:

- kubeadm is installed
- kubelet is installed (but not yet configured)
- kubectl is installed for cluster management
- Kubernetes versions are pinned and consistent

The system is now ready to initialize the cluster.
