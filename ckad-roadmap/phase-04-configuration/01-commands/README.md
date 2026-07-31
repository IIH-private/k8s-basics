# Phase 4.1 – Commands

## Goal

Learn how to override a container image's default startup behavior using `command` and `args`.

## Key Concepts

- `command` overrides the image `ENTRYPOINT`.
- `args` overrides the image `CMD`.
- `command` specifies the executable.
- `args` specifies the arguments passed to the executable.
- Both `command` and `args` are YAML lists.
- Kubernetes does not split strings by spaces.
- Use `sh -c` when multiple shell commands need to be executed.

## Examples

Override both command and arguments:

```yaml
command:
- sleep
args:
- "60"
```

Run multiple shell commands:

```yaml
command:
- sh
args:
- -c
- |
  echo "Starting..."
  pwd
  sleep 30
```

## Common Mistakes

Incorrect:

```yaml
command:
- sleep 60
```

Correct:

```yaml
command:
- sleep
args:
- "60"
```

## Useful Commands

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl delete pod <pod-name>
```

## Summary

- `command` replaces `ENTRYPOINT`.
- `args` replaces `CMD`.
- Every list item is a separate argument.
- Kubernetes never splits strings by spaces.
- `sh -c` allows execution of multiple shell commands.