# Annotations

## What are Annotations?

Annotations are metadata attached to Kubernetes objects.

Unlike labels, annotations are **not used by Kubernetes selectors**. They are intended for storing additional information used by humans or external tools.

Typical use cases include:

- Owner or contact information
- Descriptions
- Git commit IDs
- CI/CD metadata
- Build information
- Tool-specific configuration

---

## Labels vs. Annotations

| Labels | Annotations |
|--------|-------------|
| Used for identifying objects | Used for describing objects |
| Can be used by selectors | Cannot be used by selectors |
| Used by Deployments, ReplicaSets and Services | Used by people and external tools |

---

## Common Commands

Add an annotation:

    kubectl annotate pod mypod owner=Ingvar

Overwrite an existing annotation:

    kubectl annotate pod mypod owner=CKAD --overwrite

Delete an annotation:

    kubectl annotate pod mypod owner-

Show annotations:

    kubectl describe pod mypod

or

    kubectl get pod mypod -o yaml

---

## YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  annotations:
    owner: Ingvar
    description: "Frontend web server"
spec:
  containers:
  - name: nginx
    image: nginx
```

---

## Workload Objects

Deployments, DaemonSets, StatefulSets and Jobs have **two metadata levels**:

- `metadata` → annotations for the workload object itself.
- `spec.template.metadata` → annotations copied to every Pod created by the workload.

Example:

```yaml
spec:
  template:
    metadata:
      annotations:
        owner: Ingvar
```

---

## Key Points

- Annotations are stored under `metadata`.
- They provide descriptive metadata.
- They cannot be used in selectors.
- They are commonly used by CI/CD pipelines, Helm, monitoring and other Kubernetes tools.