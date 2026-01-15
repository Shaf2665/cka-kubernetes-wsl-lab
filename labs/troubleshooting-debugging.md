# CKA Lab 06 — Troubleshooting & Debugging

This lab focuses on systematic troubleshooting, which is the highest-weight skill in the CKA exam. The goal is not memorization, but learning how to think and debug under pressure.

## Lab Objectives

By the end of this lab, you will be able to:

- Troubleshoot Pods that fail to start
- Diagnose CrashLoopBackOff
- Fix ImagePullBackOff
- Debug misconfigured commands and ports
- Use the correct troubleshooting flow expected in the CKA exam

## Golden Troubleshooting Rule (CKA)

Never guess. Always observe first.

The correct order is almost always:

1. `kubectl get`
2. `kubectl describe`
3. `kubectl logs`
4. `kubectl exec`

Memorize this flow.

## Prerequisites

- A working Kubernetes cluster
- kubectl configured
- Node in Ready state

Verify:

```bash
kubectl get nodes
```

## Scenario 1 — Pod Stuck in CrashLoopBackOff

### Step 1 — Create a Broken Pod

Create a Pod manifest:

```bash
nano crashloop-pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crashloop-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "exit 1"]
```

Apply:

```bash
kubectl apply -f crashloop-pod.yaml
```

Check status:

```bash
kubectl get pod crashloop-demo
```

Expected:

```
CrashLoopBackOff
```

### Step 2 — Describe the Pod

```bash
kubectl describe pod crashloop-demo
```

Look for:

- Restart count
- Last termination reason
- Exit code

### Step 3 — Check Logs

```bash
kubectl logs crashloop-demo
```

> 📌 **CKA Insight**  
> CrashLoopBackOff usually means:
>
> - Bad command
> - Application exits immediately
> - Missing config

### Step 4 — Fix the Pod

Edit the Pod:

```bash
nano crashloop-pod.yaml
```

Change command to:

```yaml
command: ["sh", "-c", "sleep 3600"]
```

Recreate:

```bash
kubectl delete pod crashloop-demo
kubectl apply -f crashloop-pod.yaml
```

Verify:

```bash
kubectl get pod crashloop-demo
```

Expected:

```
Running
```

## Scenario 2 — ImagePullBackOff

### Step 1 — Create Pod With Invalid Image

```bash
nano imagepull-pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: imagepull-demo
spec:
  containers:
  - name: app
    image: nginx:not-a-real-tag
```

Apply:

```bash
kubectl apply -f imagepull-pod.yaml
```

Check:

```bash
kubectl get pod imagepull-demo
```

Expected:

```
ImagePullBackOff
```

### Step 2 — Describe the Pod

```bash
kubectl describe pod imagepull-demo
```

Look in Events for:

```
Failed to pull image
```

> 📌 **CKA Insight**  
> Always check:
>
> - Image name
> - Image tag
> - Registry access

### Step 3 — Fix the Image

Edit the Pod:

```yaml
image: nginx
```

Recreate:

```bash
kubectl delete pod imagepull-demo
kubectl apply -f imagepull-pod.yaml
```

Verify it reaches Running.

## Scenario 3 — Pod Runs but Application Is Not Working

### Step 1 — Create a Pod With Wrong Command

```bash
nano app-not-working.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-demo
spec:
  containers:
  - name: app
    image: nginx
    command: ["nginx"]
```

Apply:

```bash
kubectl apply -f app-not-working.yaml
```

Check status:

```bash
kubectl get pod app-demo
```

It may show Running, but nginx is not serving traffic.

### Step 2 — Inspect Logs

```bash
kubectl logs app-demo
```

> 📌 **CKA Insight**  
> Pod Running does not mean application is healthy.

### Step 3 — Fix the Command

Remove the custom command so nginx uses its default entrypoint.

Recreate the Pod without the command field.

## Scenario 4 — Pod Stuck in Pending

### Step 1 — Create a Pod With Impossible Resource Requests

```bash
nano pending-pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pending-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "10"
        memory: "10Gi"
```

Apply:

```bash
kubectl apply -f pending-pod.yaml
```

Check:

```bash
kubectl get pod pending-demo
```

Expected:

```
Pending
```

### Step 2 — Describe the Pod

```bash
kubectl describe pod pending-demo
```

Look for:

```
0/1 nodes are available: insufficient cpu, insufficient memory
```

> 📌 **CKA Insight**  
> Pending Pods are scheduler problems, not container problems.

### Step 3 — Fix Resource Requests

Reduce the requests and recreate the Pod.

## Exam-Grade Troubleshooting Checklist

When something is broken, always ask:

- Is the Pod created?
- Is it Running, Pending, or Crashing?
- What does `describe` say?
- What do logs show?
- Is this a config, image, or scheduling issue?

## Cleanup (Optional)

```bash
kubectl delete pod crashloop-demo imagepull-demo app-demo pending-demo
rm crashloop-pod.yaml imagepull-pod.yaml app-not-working.yaml pending-pod.yaml
```

## Key Takeaways (CKA-Focused)

- `kubectl describe` is your primary tool
- Logs explain crashes
- Events explain scheduling failures
- Pending ≠ broken container
- Running ≠ healthy application
- Never guess — always observe first
