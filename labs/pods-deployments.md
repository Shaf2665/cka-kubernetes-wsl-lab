# CKA Lab 01 — Pods & Deployments (Fundamentals)

This lab focuses on Pods and Deployments, which form the foundation of most workloads in Kubernetes and appear frequently in the CKA exam.

The goal is to practice:

- Creating workloads
- Observing behavior
- Performing safe changes
- Troubleshooting common issues

## Lab Objectives

By the end of this lab, you will be able to:

- Create Pods and Deployments
- Understand the relationship between Deployments, ReplicaSets, and Pods
- Scale applications
- Perform rolling updates
- Identify and troubleshoot basic failures

## Prerequisites

- A working Kubernetes cluster
- kubectl configured and working
- Node in Ready state

Verify:

```bash
kubectl get nodes
```

## Task 1 — Create a Simple Pod

Create a Pod named `nginx-pod` using the nginx image.

```bash
kubectl run nginx-pod --image=nginx
```

Verify:

```bash
kubectl get pods
```

Expected:

```
nginx-pod   Running
```

## Task 2 — Inspect the Pod

Describe the Pod:

```bash
kubectl describe pod nginx-pod
```

Check:

- Pod IP
- Node name
- Container image

View logs:

```bash
kubectl logs nginx-pod
```

## Task 3 — Delete the Pod (Important Concept)

Delete the Pod:

```bash
kubectl delete pod nginx-pod
```

Verify:

```bash
kubectl get pods
```

Observation:

- The Pod is gone
- It does not come back

> 📌 **CKA Insight**  
> Pods do not self-heal. This is why Deployments exist.

## Task 4 — Create a Deployment

Create a Deployment named `nginx-deploy` with the nginx image:

```bash
kubectl create deployment nginx-deploy --image=nginx
```

Verify:

```bash
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

## Task 5 — Scale the Deployment

Scale the Deployment to 3 replicas:

```bash
kubectl scale deployment nginx-deploy --replicas=3
```

Verify:

```bash
kubectl get pods
```

Expected:

```
3 Pods in Running state
```

> 📌 **CKA Insight**  
> Scaling is always done at the Deployment level, not the Pod level.

## Task 6 — Simulate a Pod Failure

Delete one Pod created by the Deployment:

```bash
kubectl delete pod <one-pod-name>
```

Immediately check:

```bash
kubectl get pods
```

Observation:

- A new Pod is created automatically

> 📌 **CKA Insight**  
> Deployments provide self-healing via ReplicaSets.

## Task 7 — Perform a Rolling Update

Update the image used by the Deployment:

```bash
kubectl set image deployment nginx-deploy nginx=nginx:1.25
```

Check rollout status:

```bash
kubectl rollout status deployment nginx-deploy
```

View rollout history:

```bash
kubectl rollout history deployment nginx-deploy
```

## Task 8 — Roll Back the Deployment

Roll back to the previous version:

```bash
kubectl rollout undo deployment nginx-deploy
```

Verify:

```bash
kubectl get pods
kubectl rollout history deployment nginx-deploy
```

## Task 9 — Break the Deployment (Troubleshooting Practice)

Set an invalid image:

```bash
kubectl set image deployment nginx-deploy nginx=nginx:does-not-exist
```

Check Pod status:

```bash
kubectl get pods
```

You should see:

```
ImagePullBackOff
```

Describe a failing Pod:

```bash
kubectl describe pod <pod-name>
```

> 📌 **CKA Insight**  
> Always use describe and logs before guessing.

## Task 10 — Fix the Issue

Restore a valid image:

```bash
kubectl set image deployment nginx-deploy nginx=nginx
```

Verify:

```bash
kubectl get pods
```

All Pods should return to Running.

## Cleanup (Optional)

Delete the Deployment:

```bash
kubectl delete deployment nginx-deploy
```

## Key Takeaways (CKA-Focused)

- Pods are ephemeral
- Deployments manage ReplicaSets
- Scaling and updates happen at the Deployment level
- `kubectl describe` is your best debugging tool
- Rolling updates and rollbacks are exam favorites

