# Phase 3.7 – Multi-container Pods

## Objective

Learn how to work with Pods that contain multiple containers.

Containers within the same Pod:

- share the same network namespace
- share the same Pod IP address
- communicate through `localhost`
- share data through Volumes
- start and stop together

---

## When to use Multi-container Pods

Use a Multi-container Pod when all containers belong to the same logical workload.

Typical examples:

- Application + Sidecar
- Application + Logger
- Application + Metrics Exporter
- Application + Config Generator

---

## Shared Network

Containers in the same Pod share the network namespace.

Example:

```bash
kubectl exec -it <pod> -c <container> -- wget -qO- localhost
```

No Service or DNS is required.

---

## Shared Storage

Containers can share data using a Volume.

Example:

```yaml
volumes:
- name: shared-data
  emptyDir: {}

volumeMounts:
- name: shared-data
  mountPath: /data
```

---

## emptyDir

An `emptyDir` volume is created when the Pod starts.

Properties:

- shared by all containers in the Pod
- survives container restarts
- deleted when the Pod is deleted

---

## Imperative YAML

Generate a Pod template:

```bash
kubectl run writer \
  --image=busybox:1.36 \
  --dry-run=client \
  -o yaml \
  > pod.yaml
```

Then manually add:

- volumes
- volumeMounts
- additional containers

---

## Useful Commands

Create a Pod:

```bash
kubectl apply -f pod.yaml
```

List Pods:

```bash
kubectl get pods
```

Describe a Pod:

```bash
kubectl describe pod <pod>
```

Execute a command in a specific container:

```bash
kubectl exec -it <pod> -c <container> -- sh
```

View logs from a specific container:

```bash
kubectl logs <pod> -c <container>
```

Follow logs:

```bash
kubectl logs -f <pod> -c <container>
```

Delete a Pod:

```bash
kubectl delete pod <pod>
```

---

## Key Takeaways

- A Pod can contain one or more containers.
- Multi-container is a Pod characteristic.
- Containers share the Pod IP and `localhost`.
- Containers can share Volumes.
- `emptyDir` exists only for the lifetime of the Pod.