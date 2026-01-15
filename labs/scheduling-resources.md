# CKA Lab 04 — Scheduling & Resource Management

This lab focuses on how Kubernetes schedules Pods and how resource requests and limits affect scheduling decisions. These topics are frequent in the CKA exam, especially in troubleshooting scenarios.

## Lab Objectives

By the end of this lab, you will be able to:

- Understand how the Kubernetes scheduler works (at a high level)
- Use CPU and memory requests and limits
- Observe scheduling failures due to insufficient resources
- Troubleshoot Pods stuck in Pending
- Interpret scheduler-related error messages

## Prerequisites

- A working Kubernetes cluster
- kubectl configured
- Single-node cluster with scheduling enabled

Verify:

```bash
kubectl get nodes
```

Ensure the node is in Ready state.

## Task 1 — Understand Scheduling Basics (Concept)

The Kubernetes scheduler:

- Assigns Pods to nodes
- Considers:
  - Resource requests
  - Node availability
  - Constraints (taints, selectors, affinity)

> 📌 **CKA Insight**  
> If a Pod is Pending, the scheduler could not find a suitable node.

## Task 2 — Inspect Node Capacity

Check node resources:

```bash
kubectl describe node shafiq
```

Look for:

- Capacity
- Allocatable
- CPU
- Memory

This tells you what the scheduler can work with.

## Task 3 — Create a Pod with Resource Requests

Create a Pod manifest:

```bash
nano requests-pod.yaml
```

Paste the following:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: requests-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
```

Apply:

```bash
kubectl apply -f requests-pod.yaml
```

Verify:

```bash
kubectl get pod requests-demo
```

Expected:

```
Running
```

## Task 4 — Add Resource Limits

Edit the Pod to include limits:

```bash
nano requests-pod.yaml
```

Update the container section:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "200m"
    memory: "256Mi"
```

Apply the changes:

```bash
kubectl delete pod requests-demo
kubectl apply -f requests-pod.yaml
```

Verify:

```bash
kubectl describe pod requests-demo
```

> 📌 **CKA Insight**  
>
> - Requests affect scheduling
> - Limits affect runtime behavior

## Task 5 — Create a Pod That Cannot Be Scheduled

Create a Pod with excessive resource requests:

```bash
nano unschedulable-pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: unschedulable-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "4"
        memory: "8Gi"
```

Apply:

```bash
kubectl apply -f unschedulable-pod.yaml
```

Check status:

```bash
kubectl get pod unschedulable-demo
```

Expected:

```
Pending
```

## Task 6 — Troubleshoot the Pending Pod

Describe the Pod:

```bash
kubectl describe pod unschedulable-demo
```

Look for events similar to:

```
0/1 nodes are available: insufficient cpu, insufficient memory
```

> 📌 **CKA Insight**  
> Scheduler errors always appear in the Events section.

## Task 7 — Fix the Scheduling Issue

Edit the Pod and reduce requests:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
```

Recreate the Pod:

```bash
kubectl delete pod unschedulable-demo
kubectl apply -f unschedulable-pod.yaml
```

Verify:

```bash
kubectl get pods
```

The Pod should now be Running.

## Task 8 — Optional: Observe CPU Limit Behavior

Exec into `requests-demo`:

```bash
kubectl exec -it requests-demo -- sh
```

Inside the container, run a CPU-intensive command (if available) and observe:

- CPU usage is throttled, not unlimited

> 📌 **CKA Insight**  
> CPU limits cause throttling, not termination. Memory limits cause OOMKilled.

## Cleanup (Optional)

```bash
kubectl delete pod requests-demo unschedulable-demo
rm requests-pod.yaml unschedulable-pod.yaml
```

## Key Takeaways (CKA-Focused)

- Scheduler uses requests, not limits
- Pending Pods indicate scheduling failures
- Always check `kubectl describe pod` events
- Insufficient CPU/memory is a common exam scenario
- Fixing scheduling issues is usually about adjusting requests
