# Phase 4.3 – ConfigMaps

## Goal

Learn how to create, manage, and consume ConfigMaps in Kubernetes using both imperative commands and YAML.

---

## What is a ConfigMap?

A ConfigMap stores **non-sensitive configuration data** as key/value pairs.

Typical use cases:

- Environment variables
- Configuration files
- Application settings

---

## Creating ConfigMaps

### From literal values

```bash
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=production
```

### From a file

```bash
kubectl create configmap app-config \
  --from-file=app.properties
```

Each file becomes **one key**.

### From an environment file

```bash
kubectl create configmap app-config \
  --from-env-file=app.env
```

Each line becomes **one key**.

### Declarative YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: production
```

---

## Using ConfigMaps

### Import all keys as environment variables

```yaml
envFrom:
- configMapRef:
    name: app-config
```

### Import a single key

```yaml
env:
- name: APP_COLOR
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_COLOR
```

### Mount as files

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config

volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

---

## Important Rule

One ConfigMap key becomes one file when mounted as a volume.

Example:

ConfigMap:

```yaml
data:
  APP_COLOR: blue
  APP_MODE: production
```

Mounted volume:

```text
/etc/config/
├── APP_COLOR
└── APP_MODE
```

If created with:

```bash
kubectl create configmap app-config \
  --from-file=app.properties
```

Mounted volume:

```text
/etc/app/
└── app.properties
```

---

## Verification

View ConfigMaps:

```bash
kubectl get configmaps
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

Check environment variables:

```bash
kubectl exec <pod> -- env
```

Check mounted files:

```bash
kubectl exec <pod> -- ls -l /etc/config
kubectl exec <pod> -- cat /etc/config/<filename>
```

---

## CKAD Tips

- `envFrom` imports **all** keys.
- `configMapKeyRef` imports **one** key.
- `--from-file` creates one key per file.
- `--from-env-file` creates one key per line.
- Environment variables do **not** update automatically after ConfigMap changes.
- Mounted ConfigMap files are updated automatically (after a short delay).
- `configMapKeyRef` is camelCase and case-sensitive.

---

## Imperative Commands

```bash
kubectl create configmap demo \
  --from-literal=ENV=prod

kubectl create configmap demo \
  --from-file=settings.ini

kubectl create configmap demo \
  --from-env-file=app.env
```

---

## Commands Used Most Often

```bash
kubectl get configmaps

kubectl describe configmap <name>

kubectl get configmap <name> -o yaml

kubectl exec <pod> -- env

kubectl exec <pod> -- cat <file>
```