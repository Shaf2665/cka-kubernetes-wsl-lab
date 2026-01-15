# CKA Lab 03 — Volumes & Storage (Basics)

This lab focuses on Kubernetes volumes, which are a core topic in the CKA exam. You will practice using basic volume types, understand how data persists (or does not), and troubleshoot common storage-related mistakes.

## Lab Objectives

By the end of this lab, you will be able to:

- Understand why containers need volumes
- Use emptyDir for temporary shared storage
- Use hostPath for node-level persistence
- Mount volumes into containers
- Troubleshoot common volume issues

## Prerequisites

- A working Kubernetes cluster
- kubectl configured and working
- Single-node cluster (control-plane scheduling enabled)

Verify:

```bash
kubectl get nodes
```

## Task 1 — Why Volumes Are Needed (Concept)

Containers are ephemeral:

- When a container restarts, its filesystem is reset
- Any data written inside the container is lost

Volumes solve this problem by:

- Persisting data beyond container restarts
- Allowing data sharing between containers

## Task 2 — Create a Pod with emptyDir Volume

Create a Pod manifest:

```bash
nano emptydir-pod.yaml
```

Paste the following:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: temp-storage
      mountPath: /data
  volumes:
  - name: temp-storage
    emptyDir: {}
```

Create the Pod:

```bash
kubectl apply -f emptydir-pod.yaml
```

Verify:

```bash
kubectl get pod emptydir-demo
```

## Task 3 — Write Data to emptyDir Volume

Exec into the Pod:

```bash
kubectl exec -it emptydir-demo -- sh
```

Inside the container:

```bash
echo "hello from emptyDir" > /data/message.txt
cat /data/message.txt
```

Exit the container:

```bash
exit
```

## Task 4 — Restart the Container

Delete the Pod:

```bash
kubectl delete pod emptydir-demo
```

Recreate it:

```bash
kubectl apply -f emptydir-pod.yaml
```

Exec again:

```bash
kubectl exec -it emptydir-demo -- cat /data/message.txt
```

Expected:

```
cat: can't open '/data/message.txt': No such file or directory
```

> 📌 **CKA Insight**  
> emptyDir data:
>
> - Persists only while the Pod exists
> - Is deleted when the Pod is removed

## Task 5 — Create a Pod with hostPath Volume

Create a new manifest:

```bash
nano hostpath-pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: host-storage
      mountPath: /data
  volumes:
  - name: host-storage
    hostPath:
      path: /tmp/cka-data
      type: DirectoryOrCreate
```

Create the Pod:

```bash
kubectl apply -f hostpath-pod.yaml
```

Verify:

```bash
kubectl get pod hostpath-demo
```

## Task 6 — Verify hostPath Persistence

Exec into the Pod:

```bash
kubectl exec -it hostpath-demo -- sh
```

Inside the container:

```bash
echo "hello from hostPath" > /data/message.txt
exit
```

Delete and recreate the Pod:

```bash
kubectl delete pod hostpath-demo
kubectl apply -f hostpath-pod.yaml
```

Read the file again:

```bash
kubectl exec -it hostpath-demo -- cat /data/message.txt
```

Expected:

```
hello from hostPath
```

> 📌 **CKA Insight**  
> hostPath persists data on the node filesystem.

## Task 7 — Common Volume Mistake (Wrong Mount Path)

Edit the Pod and change:

```yaml
mountPath: /data
```

to:

```yaml
mountPath: /wrongpath
```

Apply again and observe:

- Data appears "missing"
- Volume exists, but application looks in the wrong path

> 📌 **CKA Insight**  
> Most volume issues are path-related, not Kubernetes bugs.

## Task 8 — Troubleshooting Checklist (CKA-Focused)

When volume data is missing:

- Check volume type
- Check mountPath
- Check container command

Describe the Pod:

```bash
kubectl describe pod <pod-name>
```

Verify node-level paths (for hostPath)

## Cleanup (Optional)

```bash
kubectl delete pod emptydir-demo hostpath-demo
rm emptydir-pod.yaml hostpath-pod.yaml
```

## Key Takeaways (CKA-Focused)

- Containers are ephemeral by default
- emptyDir is Pod-scoped and temporary
- hostPath is node-scoped and persistent
- Volume issues are usually configuration errors
- Always verify mountPath and volume definitions
