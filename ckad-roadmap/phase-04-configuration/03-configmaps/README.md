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

This creates separate keys:

```yaml
data:
  APP_COLOR: blue
  APP_MODE: production
```

### From a file

```bash
kubectl create configmap app-config \
  --from-file=app.properties
```

The filename becomes the key.

Example:

```text
app.properties
```

becomes:

```yaml
data:
  app.properties: |
    ...
```

### From a file with a custom key

The key does not have to use the original filename.

Syntax:

```text
--from-file=<key>=<source>
```

Example:

```bash
kubectl create configmap app-config \
  --from-file=MY_CONFIG=settings.ini
```

This creates:

```yaml
data:
  MY_CONFIG: |
    ...
```

The source file is still `settings.ini`, but the ConfigMap key is `MY_CONFIG`.

### From multiple files

Multiple files can be added to the same ConfigMap:

```bash
kubectl create configmap app-config \
  --from-file=frontend.conf \
  --from-file=backend.conf
```

This creates two keys:

```text
frontend.conf
backend.conf
```

Each file becomes one ConfigMap key.

### From a directory

A directory can also be used as the source:

```bash
kubectl create configmap app-config \
  --from-file=config-files/
```

For example:

```text
config-files/
├── frontend.conf
└── backend.conf
```

creates keys:

```text
frontend.conf
backend.conf
```

### From an environment file

```bash
kubectl create configmap app-config \
  --from-env-file=app.env
```

Example `app.env`:

```text
APP_COLOR=blue
APP_MODE=production
LOG_LEVEL=info
```

This creates three separate keys:

```yaml
data:
  APP_COLOR: blue
  APP_MODE: production
  LOG_LEVEL: info
```

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

A Pod can consume ConfigMap data in different ways.

### Import all keys as environment variables

```yaml
envFrom:
- configMapRef:
    name: app-config
```

All keys from the ConfigMap become environment variables.

### Import a single key

```yaml
env:
- name: APP_COLOR
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_COLOR
```

`configMapKeyRef` selects one specific key.

The environment variable name can also be different from the ConfigMap key:

```yaml
env:
- name: DATABASE_HOST
  valueFrom:
    configMapKeyRef:
      name: db-config
      key: DB_HOST
```

---

## Mount ConfigMap as Files

A ConfigMap can be mounted into a container as a volume:

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config

containers:
- name: app
  image: alpine:3.20
  volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

### Important Rule

**One ConfigMap key becomes one file when mounted as a volume.**

Example ConfigMap:

```yaml
data:
  APP_COLOR: blue
  APP_MODE: production
```

Mounted at:

```text
/etc/config
```

produces:

```text
/etc/config/
├── APP_COLOR
└── APP_MODE
```

The files contain:

```text
/etc/config/APP_COLOR -> blue
/etc/config/APP_MODE  -> production
```

If the ConfigMap was created with:

```bash
kubectl create configmap app-config \
  --from-file=app.properties
```

the ConfigMap has one key named `app.properties`.

When mounted at `/etc/app`, the result is:

```text
/etc/app/
└── app.properties
```

---

## Select Specific Keys with `items`

By default, all ConfigMap keys are projected into the volume.

The `items` field can be used to select only specific keys.

Example ConfigMap:

```yaml
data:
  APP_COLOR: blue
  APP_MODE: production
  LOG_LEVEL: debug
```

Volume:

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
    items:
    - key: LOG_LEVEL
      path: logging-level
```

Mounted at:

```text
/etc/app
```

produces only:

```text
/etc/app/logging-level
```

with the content:

```text
debug
```

`APP_COLOR` and `APP_MODE` are not projected because they are not listed under `items`.

### `key` vs. `path`

Inside `items`:

```yaml
items:
- key: LOG_LEVEL
  path: logging-level
```

`key` selects the key from the ConfigMap:

```text
LOG_LEVEL
```

`path` determines the filename/path inside the mounted volume:

```text
logging-level
```

Therefore:

```text
ConfigMap key: LOG_LEVEL
        ↓
items.key: LOG_LEVEL
        ↓
items.path: logging-level
        ↓
mountPath: /etc/app
        ↓
/etc/app/logging-level
```

### Select multiple keys

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
    items:
    - key: APP_COLOR
      path: color
    - key: APP_MODE
      path: mode
```

Mounted at `/etc/settings`:

```text
/etc/settings/
├── color
└── mode
```

A ConfigMap key not listed under `items` is not projected into the volume.

---

## ConfigMap Runtime Updates

ConfigMaps behave differently depending on how the Pod consumes them.

### ConfigMap as Environment Variables

Example:

```yaml
env:
- name: APP_MODE
  valueFrom:
    configMapKeyRef:
      name: runtime-config
      key: APP_MODE
```

Suppose the container starts with:

```text
APP_MODE=dev
```

and the ConfigMap is later changed to:

```text
APP_MODE=prod
```

The running container still has:

```text
APP_MODE=dev
```

Environment variables are set when the container process starts.

Updating the ConfigMap does **not** automatically change the environment of an already running process.

The Pod/container must be recreated or restarted to receive the new value as an environment variable.

### ConfigMap as Mounted Volume

If the same ConfigMap is mounted as a volume:

```text
/etc/config/APP_MODE
```

and the ConfigMap changes:

```text
dev -> prod
```

Kubernetes updates the projected file after a short delay.

The running Pod does not need to be recreated for the mounted file itself to receive the updated value.

Therefore:

```text
ConfigMap update

Environment variable:
dev -> remains dev

Mounted ConfigMap file:
dev -> prod
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

Check a specific environment variable:

```bash
kubectl exec <pod> -- env | grep '^APP_MODE='
```

Check mounted files:

```bash
kubectl exec <pod> -- ls -l /etc/config
kubectl exec <pod> -- cat /etc/config/<filename>
```

---

## CKAD Tips

- `envFrom` imports **all** keys as environment variables.
- `configMapKeyRef` imports **one specific key**.
- `--from-literal` creates individual key/value pairs.
- `--from-file=<file>` uses the filename as the key.
- `--from-file=<key>=<file>` assigns a custom key name.
- Multiple `--from-file` options can be used in one command.
- `--from-file=<directory>` creates keys from files in the directory.
- `--from-env-file` creates one key per environment-file entry.
- One ConfigMap key becomes one file when mounted as a volume.
- `items` selects which ConfigMap keys are projected into a volume.
- `items[].key` selects the ConfigMap key.
- `items[].path` determines the filename/path inside the volume.
- ConfigMap environment variables do **not** update automatically in a running container.
- Mounted ConfigMap files can be updated automatically after a short delay.
- Kubernetes YAML field names such as `configMapKeyRef` are case-sensitive.

---

## Imperative Commands

Create from literal values:

```bash
kubectl create configmap demo \
  --from-literal=ENV=prod
```

Create from a file:

```bash
kubectl create configmap demo \
  --from-file=settings.ini
```

Create from a file using a custom key:

```bash
kubectl create configmap demo \
  --from-file=SETTINGS=settings.ini
```

Create from multiple files:

```bash
kubectl create configmap demo \
  --from-file=frontend.conf \
  --from-file=backend.conf
```

Create from a directory:

```bash
kubectl create configmap demo \
  --from-file=config-files/
```

Create from an environment file:

```bash
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

kubectl exec <pod> -- ls -l <mount-path>

kubectl exec <pod> -- cat <file>
```