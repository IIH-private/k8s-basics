# Phase 3.3 – Deployments

## Learning Objectives

After completing this module, I can:

- Explain the relationship between Deployment, ReplicaSet and Pod.
- Create and manage Deployments.
- Scale Deployments.
- Perform Rolling Updates and Rollbacks.
- Pause and Resume a Deployment rollout.
- Explain the difference between RollingUpdate and Recreate.
- Configure and understand `maxSurge` and `maxUnavailable`.
- Read Deployment rollout status and history.

---

# Architecture

```
Deployment
      │
      ▼
ReplicaSet
      │
      ▼
Pods
```

- Deployment manages ReplicaSets.
- ReplicaSet manages Pods.
- Pods should normally be created through a Deployment.

---

# Deployment Lifecycle

Create Deployment

↓

Deployment creates ReplicaSet

↓

ReplicaSet creates Pods

↓

Deployment continuously ensures the desired state.

---

# Creating a Deployment

Generate YAML:

```bash
kubectl create deployment nginx-deployment \
  --image=nginx:1.31 \
  --replicas=3 \
  --dry-run=client -o yaml > deployment.yaml
```

Create:

```bash
kubectl apply -f deployment.yaml
```

---

# Useful Commands

Create

```bash
kubectl apply -f deployment.yaml
```

Show Deployments

```bash
kubectl get deployments
```

Describe Deployment

```bash
kubectl describe deployment <name>
```

Scale

```bash
kubectl scale deployment <name> --replicas=5
```

Update image

```bash
kubectl set image deployment/<name> nginx=nginx:1.29.1
```

Rollout status

```bash
kubectl rollout status deployment/<name>
```

Rollout history

```bash
kubectl rollout history deployment/<name>
```

Revision details

```bash
kubectl rollout history deployment/<name> --revision=<number>
```

Rollback

```bash
kubectl rollout undo deployment/<name>
```

Rollback to specific revision

```bash
kubectl rollout undo deployment/<name> --to-revision=<number>
```

Pause rollout

```bash
kubectl rollout pause deployment/<name>
```

Resume rollout

```bash
kubectl rollout resume deployment/<name>
```

---

# RollingUpdate

Default strategy.

Goal:

- Update Pods gradually.
- Minimize downtime.
- Allow rollback.

Example:

```yaml
strategy:
  type: RollingUpdate
```

---

# Recreate

Deletes all old Pods before creating new Pods.

Example:

```yaml
strategy:
  type: Recreate
```

Advantages

- Never runs two application versions simultaneously.

Disadvantages

- Causes downtime.

---

# maxSurge

Controls how many **extra Pods** may exist during a RollingUpdate.

Example

```yaml
rollingUpdate:
  maxSurge: 1
```

If replicas = 4

Maximum Pods:

```
4 + 1 = 5
```

---

# maxUnavailable

Controls how many Pods may become unavailable during a RollingUpdate.

Example

```yaml
rollingUpdate:
  maxUnavailable: 1
```

If replicas = 4

Minimum Available:

```
4 - 1 = 3
```

---

# Labs Summary

| Lab | Configuration | Result |
|------|---------------|--------|
| 1 | maxSurge=1, maxUnavailable=1 | Default RollingUpdate |
| 2 | maxSurge=0 | No extra Pods |
| 3 | maxUnavailable=0 | No loss of Available Pods |
| 4 | maxSurge=2, maxUnavailable=2 | More aggressive rollout |
| 5 | Recreate | Temporary downtime (0 Pods) |

---

# Important Rules

- Deployment creates ReplicaSets.
- ReplicaSets create Pods.
- Only changes inside `spec.template` create a new ReplicaSet.
- Scaling does **not** create a new ReplicaSet.
- One RollingUpdate normally works between one old ReplicaSet and one new ReplicaSet.
- Old ReplicaSets are kept for rollback.
- Rollback reuses an existing ReplicaSet when possible.

---

# Pause / Resume

Pause allows multiple Deployment changes without starting a rollout.

```
pause

↓

change image

↓

change env

↓

change resources

↓

resume

↓

one single rollout
```

---

# CKAD Tips

Know these commands:

```bash
kubectl create deployment
kubectl scale deployment
kubectl set image
kubectl rollout status
kubectl rollout history
kubectl rollout undo
kubectl rollout pause
kubectl rollout resume
```

Know the difference between:

- Deployment
- ReplicaSet
- Pod

Understand:

- RollingUpdate
- Recreate
- maxSurge
- maxUnavailable

---

# Key Takeaways

- Deployment is the recommended way to manage Pods.
- Deployments create ReplicaSets.
- ReplicaSets create Pods.
- RollingUpdate minimizes downtime.
- Recreate removes all old Pods before creating new ones.
- `maxSurge` controls extra Pods.
- `maxUnavailable` controls minimum available Pods.
- Rollbacks are fast because old ReplicaSets are preserved.