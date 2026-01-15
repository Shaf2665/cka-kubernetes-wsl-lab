# 01 — WSL2 Setup for Kubernetes (CKA Lab)

This document explains how to prepare **Windows Subsystem for Linux (WSL2)**
to run Kubernetes using **kubeadm**.  
The goal is to create a **stable Linux environment** suitable for **CKA
exam preparation**, not a production setup.

---

## Why WSL2?

WSL2 provides:
- A real Linux kernel
- systemd support
- Good performance for local labs

This makes it a reasonable choice for learning Kubernetes fundamentals
on a Windows machine.

---

## Prerequisites

- Windows 10 (version 2004+) or Windows 11
- Administrator access on Windows
- Internet connectivity

---

## Step 1 — Enable WSL2

Open **PowerShell as Administrator** and run:

```powershell
wsl --install

