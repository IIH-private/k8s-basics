# 02 – ReplicaSets

## Learning Objectives

After completing this module, you should be able to:

- Explain why ReplicaSets exist.
- Create a ReplicaSet from a YAML manifest.
- Understand the relationship between `replicas`, `selector`, and `template`.
- Scale ReplicaSets up and down.
- Explain Kubernetes self-healing.
- Troubleshoot ReplicaSets using `kubectl describe`.
- Understand the relationship between Pods, ReplicaSets, and Deployments.

---

# What is a ReplicaSet?

A ReplicaSet is a Kubernetes controller that ensures a specified number of identical Pods are always running.

Unlike a Pod, a ReplicaSet continuously monitors the cluster and compares:

- Desired state
- Current state

If they differ, Kubernetes automatically reconciles the difference.

Example:

Desired Pods:

```
3
```

Current Pods:

```
2
```

ReplicaSet automatically creates one new Pod.

---

# Why do ReplicaSets exist?

A Pod by itself is **not self-healing**.

If a Pod is deleted:

```bash
kubectl delete pod nginx
```

the Pod disappears permanently.

ReplicaSet solves this problem.

Instead of saying:

> Create one Pod.

you declare:

> I always want three Pods running.

ReplicaSet continuously enforces this desired state.

---

# ReplicaSet Architecture

```
ReplicaSet
      │
      ▼
 Desired Pods = 3
      │
 ┌────┼────┐
 ▼    ▼    ▼
Pod  Pod  Pod
```

If one Pod disappears:

```
ReplicaSet
      │
Desired = 3
Current = 2
      │
      ▼
Create new Pod
```

---

# ReplicaSet Manifest

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx-rs

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:1.27
```

---

# Manifest Explanation

## metadata

Describes the ReplicaSet object itself.

```yaml
metadata:
  name: nginx-rs
```

---

## replicas

Defines the desired number of Pods.

```yaml
replicas: 3
```

ReplicaSet continuously ensures this number exists.

---

## selector

Determines which Pods belong to the ReplicaSet.

```yaml
selector:
  matchLabels:
    app: nginx
```

ReplicaSet never uses Pod names.

It always identifies Pods by labels.

---

## template

Defines how new Pods should be created.

```yaml
template:
```

This is essentially a Pod manifest embedded inside the ReplicaSet.

Notice that the template has:

- metadata
- spec

but **not**

- apiVersion
- kind
- metadata.name

ReplicaSet automatically generates unique Pod names.

---

# Selector and Template

One of the most important ReplicaSet rules:

```
selector.matchLabels
MUST match
template.metadata.labels
```

Correct:

```yaml
selector:
  matchLabels:
    app: nginx

template:
  metadata:
    labels:
      app: nginx
```

Incorrect:

```yaml
selector:
  matchLabels:
    app: nginx

template:
  metadata:
    labels:
      app: apache
```

Kubernetes rejects this configuration because the ReplicaSet would never find its own Pods.

---

# Labels

ReplicaSet identifies Pods using labels.

Example:

```yaml
labels:
  app: nginx
```

Extra labels are allowed.

Example:

```yaml
labels:
  app: nginx
  env: production
  version: v1
```

This still matches:

```yaml
matchLabels:
  app: nginx
```

`matchLabels` always uses **AND** logic.

---

# Self-Healing

Deleting a Pod:

```bash
kubectl delete pod <pod-name>
```

ReplicaSet immediately notices:

```
Desired = 3

Current = 2
```

and creates a replacement Pod automatically.

The replacement Pod receives:

- a new name
- a new UID
- typically a new IP address

Pods are disposable.

---

# Scaling

Increase replicas:

```bash
kubectl scale rs nginx-rs --replicas=5
```

Decrease replicas:

```bash
kubectl scale rs nginx-rs --replicas=2
```

ReplicaSet automatically creates or removes Pods until the desired number is reached.

---

# Imperative vs Declarative

Imperative:

```bash
kubectl scale rs nginx-rs --replicas=5
```

Changes the running object inside the cluster.

Declarative:

Modify:

```yaml
replicas: 5
```

Then apply:

```bash
kubectl apply -f manifests/rs-nginx.yaml
```

The YAML manifest becomes the source of truth.

---

# Useful Commands

Create ReplicaSet

```bash
kubectl apply -f manifests/rs-nginx.yaml
```

List ReplicaSets

```bash
kubectl get rs
```

List Pods

```bash
kubectl get pods
```

Describe ReplicaSet

```bash
kubectl describe rs nginx-rs
```

Scale ReplicaSet

```bash
kubectl scale rs nginx-rs --replicas=5
```

Delete a Pod

```bash
kubectl delete pod <pod-name>
```

Validate YAML

```bash
kubectl apply --dry-run=client -f manifests/rs-nginx.yaml
```

---

# Troubleshooting

Useful commands:

```bash
kubectl describe rs nginx-rs
```

Pay attention to:

- Selector
- Replicas
- Pod Status
- Events

Events often explain why Pods were or were not created.

---

# Pod vs ReplicaSet

| Pod | ReplicaSet |
|------|------------|
| Creates one Pod | Maintains multiple Pods |
| No self-healing | Self-healing |
| No scaling | Supports scaling |
| No controller | Kubernetes controller |
| Pods managed manually | Pods managed automatically |

---

# Best Practices

- Always use labels consistently.
- Ensure selector matches template labels.
- Treat Pods as disposable.
- Prefer declarative manifests in production.
- Use `kubectl describe` during troubleshooting.

---

# Key Takeaways

- ReplicaSet maintains the desired number of Pods.
- ReplicaSet uses labels, never Pod names.
- Pods are recreated automatically when deleted.
- Scaling changes the desired state.
- `selector` and `template.labels` must match.
- ReplicaSets are usually managed indirectly through Deployments.