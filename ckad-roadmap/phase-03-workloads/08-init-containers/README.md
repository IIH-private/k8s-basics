# Phase 3.8 – Init Containers

## Goal

Learn how to prepare a Pod before the application containers start by using Init Containers.

---

## What is an Init Container?

An Init Container is a special container that runs **before** the regular application containers.

It is typically used to:

- Wait for a dependency (database, service, etc.)
- Generate configuration files
- Download data
- Initialize directories
- Prepare shared volumes

Application containers start **only after all Init Containers complete successfully**.

---

## Execution Order

```text
Init Container 1
      ↓
Init Container 2
      ↓
Application Container(s)
```

- Init Containers run sequentially.
- Each Init Container must exit successfully (`exit 0`).
- If one Init Container fails, the Pod will not start the application containers.

---

## YAML Structure

```yaml
spec:
  initContainers:
  - name: init
    image: busybox
    command:
    - sh
    - -c
    - echo "Initializing..."

  containers:
  - name: app
    image: nginx
```

`initContainers` and `containers` are sibling fields under `spec`.

---

## Sharing Data

The most common pattern is using a shared `emptyDir` volume.

```text
           emptyDir
        ┌──────────────┐
        │ index.html   │
        └──────────────┘
             ▲      ▲
             │      │
        Init        App
       writes      reads
```

Example:

```yaml
volumes:
- name: shared-data
  emptyDir: {}

initContainers:
- name: init
  image: busybox
  volumeMounts:
  - name: shared-data
    mountPath: /work

containers:
- name: app
  image: nginx
  volumeMounts:
  - name: shared-data
    mountPath: /usr/share/nginx/html
```

---

## Useful Commands

Create resources:

```bash
kubectl apply -f pod.yaml
```

Watch Pod status:

```bash
kubectl get pods -w
```

Describe the Pod:

```bash
kubectl describe pod <pod-name>
```

View Init Container logs:

```bash
kubectl logs <pod-name> -c <init-container-name>
```

View application container logs:

```bash
kubectl logs <pod-name> -c <container-name>
```

Execute a command inside the application container:

```bash
kubectl exec -it <pod-name> -c <container-name> -- sh
```

---

## Troubleshooting

Recommended troubleshooting workflow:

```text
kubectl get pods
        ↓
kubectl describe pod <pod>
        ↓
kubectl logs <pod> -c <init-container>
```

Look for:

- `Init:0/1`
- `Init:Error`
- `Init:CrashLoopBackOff`
- Exit Code
- Events
- Container logs

---

## CKAD Tips

- Init Containers run only once.
- They always run sequentially.
- Application containers wait until all Init Containers finish successfully.
- Use `emptyDir` to share data between Init Containers and application containers.
- Remember to specify the container name when viewing logs:

```bash
kubectl logs <pod> -c <container-name>
```

---

## Key Takeaways

- Init Containers prepare the Pod before the application starts.
- Failed Init Containers prevent application containers from starting.
- Shared volumes are the standard way to exchange data.
- `kubectl describe` and `kubectl logs` are the primary troubleshooting tools.