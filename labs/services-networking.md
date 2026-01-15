# CKA Lab 02 — Services & Networking (Fundamentals)

This lab focuses on Kubernetes Services and basic networking, a high-weight area in the CKA exam. You will practice creating Services, understanding selectors and endpoints, and troubleshooting common connectivity issues.

## Lab Objectives

By the end of this lab, you will be able to:

- Expose applications using Services
- Understand ClusterIP and NodePort
- Verify Service selectors and Endpoints
- Troubleshoot Services that are not routing traffic

## Prerequisites

- A working Kubernetes cluster
- kubectl configured and working
- Control-plane taint removed (single-node lab)

Verify:

```bash
kubectl get nodes
```

## Task 1 — Create a Deployment for Testing

Create a Deployment named `web` using the nginx image:

```bash
kubectl create deployment web --image=nginx
```

Scale it to 2 replicas:

```bash
kubectl scale deployment web --replicas=2
```

Verify:

```bash
kubectl get pods -l app=web
```

## Task 2 — Create a ClusterIP Service

Expose the Deployment internally using a ClusterIP Service:

```bash
kubectl expose deployment web --port=80 --target-port=80 --name=web-svc
```

Verify Service creation:

```bash
kubectl get svc web-svc
```

Expected type:

```
TYPE: ClusterIP
```

## Task 3 — Inspect the Service and Endpoints

Describe the Service:

```bash
kubectl describe svc web-svc
```

Check:

- Selector
- TargetPort
- Endpoints section

Verify Endpoints explicitly:

```bash
kubectl get endpoints web-svc
```

Expected:

```
One endpoint per Pod IP
```

> 📌 **CKA Insight**  
> If a Service has no endpoints, it cannot route traffic.

## Task 4 — Test Service Connectivity from Inside the Cluster

Run a temporary test Pod:

```bash
kubectl run test-client --image=busybox --restart=Never -- sleep 3600
```

Exec into the Pod:

```bash
kubectl exec -it test-client -- sh
```

Inside the Pod, test the Service:

```bash
wget -qO- http://web-svc
```

You should see HTML output from nginx.

Exit the Pod:

```bash
exit
```

## Task 5 — Convert to NodePort Service

Expose the Deployment using NodePort:

```bash
kubectl expose deployment web \
  --type=NodePort \
  --port=80 \
  --target-port=80 \
  --name=web-nodeport
```

Verify:

```bash
kubectl get svc web-nodeport
```

Note the assigned NodePort (e.g. 300xx).

## Task 6 — Access the Application via NodePort

Get the node IP:

```bash
kubectl get nodes -o wide
```

Use the node IP and NodePort:

```bash
curl http://<NODE-IP>:<NODE-PORT>
```

You should receive nginx output.

> 📌 **CKA Insight**  
> NodePort exposes services externally but is rarely used in production.

## Task 7 — Break the Service (Selector Mismatch)

Edit the Service:

```bash
kubectl edit svc web-svc
```

Change the selector from:

```yaml
selector:
  app: web
```

to:

```yaml
selector:
  app: wrong-label
```

Save and exit.

Verify endpoints:

```bash
kubectl get endpoints web-svc
```

Expected:

```
<none>
```

## Task 8 — Troubleshoot the Broken Service

Attempt access again from test-client:

```bash
kubectl exec -it test-client -- wget -qO- http://web-svc
```

It will fail.

Describe the Service:

```bash
kubectl describe svc web-svc
```

> 📌 **CKA Insight**  
> Service issues are usually caused by:
>
> - Wrong selector
> - Missing labels
> - Pods not running

## Task 9 — Fix the Service

Edit the Service and restore the correct selector:

```yaml
selector:
  app: web
```

Verify endpoints:

```bash
kubectl get endpoints web-svc
```

Test again from the client Pod.

## Task 10 — Cleanup (Optional)

```bash
kubectl delete svc web-svc web-nodeport
kubectl delete deployment web
kubectl delete pod test-client
```

## Key Takeaways (CKA-Focused)

- Services route traffic using selectors
- Endpoints show the actual backing Pods
- No endpoints = no traffic
- ClusterIP is internal-only
- NodePort exposes services externally
- `kubectl describe svc` is critical for troubleshooting
