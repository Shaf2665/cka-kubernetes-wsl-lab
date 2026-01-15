# 01 — WSL2 Setup for Kubernetes (CKA Lab)

This document explains how to prepare Windows Subsystem for Linux (WSL2) to run Kubernetes using kubeadm. The goal is to create a stable Linux environment suitable for CKA exam preparation, not a production setup.

## Why WSL2?

WSL2 provides:

- A real Linux kernel
- systemd support
- Good performance for local labs

This makes it a reasonable choice for learning Kubernetes fundamentals on a Windows machine.

## Prerequisites

- Windows 10 (version 2004+) or Windows 11
- Administrator access on Windows
- Internet connectivity

## Step 1 — Enable WSL2

Open PowerShell as Administrator and run:

```powershell
wsl --install
```

This command:

- Enables required Windows features
- Installs WSL
- Sets WSL2 as the default version

Restart your system when prompted.

## Step 2 — Verify WSL Version

After reboot, open PowerShell and verify:

```powershell
wsl --status
```

You should see:

```
Default Version: 2
```

If not, set it explicitly:

```powershell
wsl --set-default-version 2
```

## Step 3 — Install Ubuntu on WSL

Install Ubuntu from the Microsoft Store (Ubuntu 22.04 LTS is recommended).

After installation:

- Launch Ubuntu
- Create a Linux username and password

## Step 4 — Update the Linux System

Inside Ubuntu, run:

```bash
sudo apt update && sudo apt upgrade -y
```

This ensures all base packages are up to date.

## Step 5 — Enable systemd (Important)

Kubernetes components rely on systemd. WSL does not enable it by default.

Edit the WSL configuration file:

```bash
sudo nano /etc/wsl.conf
```

Add the following content:

```ini
[boot]
systemd=true
```

Save and exit.

Restart WSL from Windows:

```powershell
wsl --shutdown
```

Reopen Ubuntu.

Verify systemd is running:

```bash
ps -p 1 -o comm=
```

Expected output:

```
systemd
```

## Step 6 — Basic System Preparation

These steps prevent common kubeadm failures later.

### Disable Swap (Required)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Verify:

```bash
free -h
```

Swap should show 0B.

### Enable Required Kernel Parameters

```bash
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF

sudo sysctl --system
```

Verify:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected output:

```
1
```

## Result

At this point, you have:

- WSL2 correctly configured
- Ubuntu running with systemd
- Swap disabled
- Kernel and networking prepared for Kubernetes

You are now ready to install a container runtime.
