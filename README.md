# CKA Kubernetes WSL Lab

This repository documents the setup of a **single-node Kubernetes cluster**
using **kubeadm on WSL2 (Ubuntu)**, created as part of my **Certified Kubernetes
Administrator (CKA)** exam preparation.

The goal of this project is to build a **clean, exam-aligned Kubernetes
environment** without relying on tools like minikube or managed platforms,
while keeping the setup simple and reproducible.

---

## Why This Repository?

- To understand Kubernetes installation using **kubeadm**
- To practice **CKA-relevant concepts** in a real cluster environment
- To document the setup clearly for revision and reference
- To share a practical learning lab with others preparing for CKA

---

## Environment

- Host OS: Windows
- Linux Environment: WSL2
- Distribution: Ubuntu
- Container Runtime: containerd
- Kubernetes Version: v1.34.x
- Cluster Type: Single-node (control-plane also schedules workloads)

---

## What This Repository Covers

- WSL2 preparation for Kubernetes
- Installing and configuring containerd
- Installing Kubernetes components (kubeadm, kubelet, kubectl)
- Initializing a single-node cluster with kubeadm
- Basic CKA-oriented labs:
  - Pods and Deployments
  - Services and Networking
  - Troubleshooting common issues

---

## What This Repository Does NOT Cover

- Multi-node cluster setup
- Cloud-managed Kubernetes services
- Production hardening

These topics are intentionally excluded to keep the focus on **CKA fundamentals**.

---

## Getting Started

Follow the documentation in the `setup/` directory in order:

1. WSL setup
2. Container runtime installation
3. Kubernetes installation
4. Cluster initialization

---

## Disclaimer

This setup is intended for **learning and exam preparation purposes only**.
Behavior on WSL may differ from production or cloud environments.

---

## License

MIT

