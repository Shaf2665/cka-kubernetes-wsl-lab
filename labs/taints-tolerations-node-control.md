# CKA Lab 05 — Taints, Tolerations & Node Control

This lab focuses on controlling where Pods can (and cannot) run using taints, tolerations, and node control commands. These topics are very common in the CKA exam, especially in troubleshooting and maintenance scenarios.

## Lab Objectives

By the end of this lab, you will be able to:

- Understand what taints and tolerations are
- Apply and remove taints from nodes
- Configure Pods with tolerations
- Use cordon, uncordon, and drain
- Safely manage nodes during maintenance

## Prerequisites

- A working Kubernetes cluster
- kubectl configured
- Single-node cluster (control-plane scheduling enabled)

Verify:

```bash
kubectl get nodes
```

Ensure the node is Ready.

## Task 1 — Understand Taints & Tolerations (Concept)

- **Taint**: Applied to a node, repels Pods
- **Toleration**: Applied to a Pod, allows it to be scheduled on a tainted node

> 📌 **CKA Insight**  
> Taints are set on nodes. Tolerations are set on Pods. They do not guarantee placement; they only allow it.

## Task 2 — View Existing Taints

Check current taints on the node:

```bash
kubectl describe node shafiq | grep Taint
```

In a single-node lab, this should normally show no taints.

## Task 3 — Apply a Taint to the Node

Apply a taint that prevents Pods from scheduling:

```bash
kubectl taint node shafiq env=prod:NoSchedule
```

Verify:

```bash
kubectl describe node shafiq | grep Taint
```

Expected:

```
env=prod:NoSchedule
```

## Task 4 — Create a Pod Without Toleration

Create a Pod manifest:

```bash
nano no-toleration-pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-toleration
spec:
  containers:
  - name: app
    image: nginx
```

Apply:

```bash
kubectl apply -f no-toleration-pod.yaml
```

Check status:

```bash
kubectl get pod no-toleration
```

Expected:

```
Pending
```

Describe the Pod:

```bash
kubectl describe pod no-toleration
```

Look for:

```
had taint {env=prod: NoSchedule}
```

## Task 5 — Create a Pod With Toleration

Create a Pod that tolerates the taint:

```bash
nano toleration-pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-toleration
spec:
  tolerations:
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"
  containers:
  - name: app
    image: nginx
```

Apply:

```bash
kubectl apply -f toleration-pod.yaml
```

Verify:

```bash
kubectl get pods
```

Expected:

- `with-toleration` → Running
- `no-toleration` → Pending

> 📌 **CKA Insight**  
> Tolerations allow scheduling but do not force it.

## Task 6 — Remove the Taint

Remove the taint from the node:

```bash
kubectl taint node shafiq env=prod:NoSchedule-
```

Verify:

```bash
kubectl describe node shafiq | grep Taint
```

(No output indicates success.)

Check Pods:

```bash
kubectl get pods
```

The previously pending Pod should now become Running.

## Task 7 — Cordon the Node

Mark the node as unschedulable:

```bash
kubectl cordon shafiq
```

Verify:

```bash
kubectl get nodes
```

Expected:

```
SchedulingDisabled
```

Create a new Pod:

```bash
kubectl run cordon-test --image=nginx
```

Check status:

```bash
kubectl get pod cordon-test
```

Expected:

```
Pending
```

## Task 8 — Uncordon the Node

Allow scheduling again:

```bash
kubectl uncordon shafiq
```

Verify:

```bash
kubectl get nodes
```

The Pod should now be scheduled and running.

## Task 9 — Drain the Node (Concept + Command)

Draining a node:

- Evicts Pods
- Used during maintenance
- Respects PodDisruptionBudgets

Run (dry understanding):

```bash
kubectl drain shafiq --ignore-daemonsets --delete-emptydir-data
```

> 📌 **CKA Insight**  
> In single-node clusters, draining will evict almost everything. Understand the command; use carefully.

After understanding, bring node back:

```bash
kubectl uncordon shafiq
```

## Cleanup (Optional)

```bash
kubectl delete pod no-toleration with-toleration cordon-test
rm no-toleration-pod.yaml toleration-pod.yaml
```

## Key Takeaways (CKA-Focused)

- Taints repel Pods
- Tolerations allow Pods to be scheduled
- NoSchedule prevents new Pods
- `cordon` stops scheduling
- `uncordon` resumes scheduling
- `drain` safely prepares nodes for maintenance
