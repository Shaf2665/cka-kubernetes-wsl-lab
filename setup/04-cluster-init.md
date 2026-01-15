# 04 — Kubernetes Cluster Initialization (kubeadm init)

This document explains how to initialize a single-node Kubernetes cluster using kubeadm on WSL2. The setup is intentionally simple and aligned with CKA exam preparation.

## Objective

By the end of this step, you will have:

- A working Kubernetes control-plane node
- kubectl configured for the current user
- A pod network installed
- A usable single-node cluster for CKA labs

## Prerequisites

Before proceeding, ensure:

- WSL2 is configured and systemd is running
- containerd is installed and running
- kubeadm, kubelet, and kubectl are installed
- Swap is disabled

(Completed in previous steps.)

## Step 1 — Initialize the Cluster

Run the following command to initialize the Kubernetes control plane:

```bash
sudo kubeadm init
```

Notes:

- kubeadm automatically detects the installed Kubernetes version
- Certificates, control-plane components, and configuration files are created
- This step may take a few minutes

## Step 2 — Configure kubectl Access

After a successful initialization, kubeadm prints instructions to configure kubectl. Run the following commands as your normal user:

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify access:

```bash
kubectl get nodes
```

Expected output:

```
NAME     STATUS     ROLES           AGE   VERSION
shafiq   NotReady   control-plane   ...
```

NotReady is expected at this stage because networking is not installed yet.

## Step 3 — Install Pod Network (CNI)

Kubernetes requires a Container Network Interface (CNI) plugin to allow pods to communicate.

For this lab, we use Calico, which is stable and commonly used.

Install Calico:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Wait for system pods to start:

```bash
kubectl get pods -n kube-system
```

All pods should reach the Running state.

Verify node status:

```bash
kubectl get nodes
```

Expected output:

```
NAME     STATUS   ROLES           AGE   VERSION
shafiq   Ready    control-plane   ...
```

## Step 4 — Allow Scheduling on Control Plane (Single-Node Only)

By default, control-plane nodes do not run application workloads. For a single-node lab, remove the control-plane taint:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

Verify taints are removed:

```bash
kubectl describe node shafiq | grep Taint
```

(No output indicates success.)

## Step 5 — Final Verification

Run the following checks:

```bash
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

You should see:

- Node in Ready state
- All system pods running
- Kubernetes API server reachable

## Result

You now have:

- A fully functional single-node Kubernetes cluster
- kubeadm-based setup (exam-aligned)
- kubectl configured for daily practice
- A stable environment for CKA labs

## What's Next?

You are now ready to begin CKA-oriented practice labs.

Suggested order:

- Pods and Deployments
- Services and Networking
- Volumes
- Scheduling basics
- Troubleshooting scenarios

These labs will be added under the `labs/` directory.

## Important Disclaimer

This cluster is intended for learning and exam preparation only. Some behaviors on WSL may differ from real production or cloud environments.
