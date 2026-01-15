# 02 — Container Runtime Setup (containerd)

This document explains how to install and configure containerd as the container runtime for Kubernetes in a WSL2-based single-node cluster. The configuration is aligned with kubeadm and CKA exam expectations.

## Why containerd?

Kubernetes does not run containers directly. It relies on a container runtime that implements the CRI (Container Runtime Interface).

containerd is:

- CNCF graduated
- Lightweight and stable
- Default runtime used by many Kubernetes distributions
- Fully supported by kubeadm

For CKA preparation, containerd is the recommended choice.

## Prerequisites

Before proceeding, ensure:

- WSL2 is installed and configured
- Ubuntu is running with systemd enabled
- Swap is disabled
- Kernel parameters for Kubernetes are applied

(These were completed in `01-wsl-setup.md`.)

## Step 1 — Install containerd

Update package index and install containerd:

```bash
sudo apt update
sudo apt install -y containerd
```

Verify installation:

```bash
containerd --version
```

You should see a valid version output.

## Step 2 — Generate Default containerd Configuration

Kubernetes expects containerd to use a specific configuration. Generate the default config file:

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

## Step 3 — Configure systemd Cgroups (Critical)

Kubernetes requires containerd to use systemd cgroups.

Edit the configuration file:

```bash
sudo nano /etc/containerd/config.toml
```

Find the following line:

```toml
SystemdCgroup = false
```

Change it to:

```toml
SystemdCgroup = true
```

Save and exit the editor.

## Step 4 — Restart and Enable containerd

Apply the configuration and ensure containerd starts automatically:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Verify containerd status:

```bash
systemctl status containerd --no-pager
```

Expected state:

```
Active: active (running)
```

## Step 5 — Verify CRI Functionality

Kubernetes communicates with the runtime using CRI. We verify this indirectly at this stage.

Check that the containerd socket exists:

```bash
ls -l /run/containerd/containerd.sock
```

The file should exist.

## Result

At this point:

- containerd is installed
- systemd cgroups are enabled
- containerd is running and managed by systemd
- The CRI socket is available for Kubernetes

Your system is now ready to install Kubernetes components.
