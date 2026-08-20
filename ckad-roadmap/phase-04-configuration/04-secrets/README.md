# Phase 4.4 – Secrets

## Overview

Kubernetes Secrets are used to store and provide sensitive configuration data such as:

- usernames
- passwords
- API keys
- tokens
- TLS certificates
- private registry credentials

A Secret is similar to a ConfigMap, but it is intended for sensitive data.

> Important: Values stored under `data` are Base64 encoded. Base64 is encoding, not encryption.

---

## 1. Generic / Opaque Secrets

A normal generic Secret typically has the type:

    type: Opaque

Create a generic Secret from literal values:

    kubectl create secret generic database-secret \
      --from-literal=DB_USER=appuser \
      --from-literal=DB_PASSWORD=SuperSecret123

Inspect it:

    kubectl get secret database-secret

    kubectl get secret database-secret -o yaml

---

## 2. Generate a Secret Manifest Imperatively

Use `--dry-run=client -o yaml` to generate YAML without creating the Secret:

    kubectl create secret generic app-secret \
      --from-literal=APP_USER=ckad \
      --from-literal=APP_PASSWORD=secret123 \
      --dry-run=client -o yaml

Save it directly to a file:

    kubectl create secret generic app-secret \
      --from-literal=APP_USER=ckad \
      --from-literal=APP_PASSWORD=secret123 \
      --dry-run=client -o yaml > app-secret.yaml

Then create it:

    kubectl apply -f app-secret.yaml

---

## 3. `data` vs `stringData`

### `data`

Values under `data` must be Base64 encoded:

    apiVersion: v1
    kind: Secret
    metadata:
      name: app-secret
    type: Opaque
    data:
      APP_USER: Y2thZA==

Decode a Base64 value:

    echo 'Y2thZA==' | base64 -d; echo

### `stringData`

`stringData` allows plain-text values in the input manifest:

    apiVersion: v1
    kind: Secret
    metadata:
      name: app-secret
    type: Opaque
    stringData:
      APP_USER: ckad
      APP_PASSWORD: secret123

Kubernetes converts the values when the Secret is stored.

### Remember

    data       -> Base64 encoded values
    stringData -> plain-text input

Base64 is NOT encryption.

---

## 4. Read and Decode a Secret Value

Get the Base64 encoded value:

    kubectl get secret database-secret \
      -o jsonpath='{.data.DB_PASSWORD}'

Decode it:

    kubectl get secret database-secret \
      -o jsonpath='{.data.DB_PASSWORD}' | base64 -d; echo

If a key contains a dot, escape it in JSONPath:

    kubectl get secret file-secret \
      -o jsonpath='{.data.database-password\.txt}' | base64 -d; echo

---

## 5. Create Secrets from Files

### `--from-file=file`

Example:

    kubectl create secret generic file-secret \
      --from-file=database-password.txt

If the source file is:

    database-password.txt

the Secret contains one key:

    database-password.txt

The value is the complete content of the file.

When mounted as a Secret volume, the key becomes the filename.

---

### `--from-file=KEY=file`

A custom Secret key can be specified:

    kubectl create secret generic file-secret \
      --from-file=DB_PASSWORD=database-password.txt

Result:

    Source file: database-password.txt
            |
            v
    Secret key: DB_PASSWORD
            |
            v
    Mounted file: DB_PASSWORD

The filename of the source file does not have to match the Secret key.

---

## 6. Create a Secret from an Environment File

Assume `database.env` contains:

    DB_USER=postgres
    DB_PASSWORD=PostgresSecret123
    DB_NAME=production

Create the Secret:

    kubectl create secret generic database-env-secret \
      --from-env-file=database.env

This creates three Secret keys:

    DB_USER
    DB_PASSWORD
    DB_NAME

### Important difference

`--from-file`:

    --from-file=app.env

creates one Secret key:

    app.env

The value is the entire file content.

`--from-file=KEY=file`:

    --from-file=CONFIG=app.env

creates one Secret key:

    CONFIG

The value is the entire file content.

`--from-env-file`:

    --from-env-file=app.env

parses the `KEY=value` lines and creates individual Secret keys.

---

## 7. Inject All Secret Keys with `envFrom`

Use `envFrom.secretRef` when all Secret keys should become environment variables:

    envFrom:
      - secretRef:
          name: app-secret

If the Secret contains:

    APP_USER=admin
    APP_PASSWORD=secret123

the container receives:

    APP_USER=admin
    APP_PASSWORD=secret123

The Secret key names become the environment variable names.

---

## 8. Inject One Secret Key with `secretKeyRef`

Use `secretKeyRef` when a specific Secret key should be used:

    env:
      - name: DATABASE_PASSWORD
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: APP_PASSWORD

The environment variable name does not have to match the Secret key name:

    APP_PASSWORD -> DATABASE_PASSWORD

Use `secretKeyRef` when individual keys need to be selected or renamed.

---

## 9. Mount a Secret as Files

A Secret can be mounted as a volume:

    spec:
      containers:
        - name: app
          image: alpine
          volumeMounts:
            - name: secret-volume
              mountPath: /etc/secrets
              readOnly: true

      volumes:
        - name: secret-volume
          secret:
            secretName: app-secret

If the Secret contains:

    APP_USER
    APP_PASSWORD

the container receives:

    /etc/secrets/APP_USER
    /etc/secrets/APP_PASSWORD

Rule:

    Secret key   -> filename
    Secret value -> file content

`/etc/secrets` is not a Kubernetes default path. The path is chosen with `mountPath`.

---

## 10. Select Secret Keys with `items`

`items` can be used to mount only selected keys:

    volumes:
      - name: secret-volume
        secret:
          secretName: database-secret
          items:
            - key: DB_USER
              path: username
            - key: DB_PASSWORD
              path: password

With:

    mountPath: /etc/database

the result is:

    /etc/database/username
    /etc/database/password

Keys not included under `items` are not mounted.

### `path`

`path` specifies the filename or relative path created under the volume mount.

Example:

    key: DB_PASSWORD
    path: password

means:

    Secret key DB_PASSWORD
             |
             v
    /etc/database/password

The file contains the Secret VALUE, not a `key=value` pair.

---

## 11. Secret File Permissions with `defaultMode`

Set the default permissions for files created from a Secret:

    volumes:
      - name: secret-volume
        secret:
          secretName: database-secret
          defaultMode: 0440

Common Linux permission values:

    0400 -> r--------  owner: read
    0600 -> rw-------  owner: read + write
    0440 -> r--r-----  owner/group: read
    0644 -> rw-r--r--  owner: read/write, group/others: read

`defaultMode` applies to all projected Secret files unless overridden for an individual item.

---

## 12. Override Permissions for an Individual Item

An individual item can override `defaultMode`:

    volumes:
      - name: secret-volume
        secret:
          secretName: database-secret
          defaultMode: 0440
          items:
            - key: DB_USER
              path: username
            - key: DB_PASSWORD
              path: password
              mode: 0400

Result:

    username -> 0440
    password -> 0400

Rule:

    defaultMode -> default for Secret files
    item.mode   -> overrides defaultMode for that item

---

## 13. `defaultMode` vs `readOnly`

These settings control different things.

### `defaultMode`

Controls Linux permissions on the projected Secret files:

    defaultMode: 0400

### `readOnly`

Controls whether the volume mount is writable from the container:

    volumeMounts:
      - name: secret-volume
        mountPath: /etc/secrets
        readOnly: true

Remember:

    defaultMode -> file permissions
    readOnly    -> volume mount behavior

Even if a file mode contains write permission, a read-only mount prevents writes through that mount.

---

## 14. Secret Volumes Use Symbolic Links

Kubernetes Secret volume files are normally exposed through symbolic links.

For example:

    ls -l /etc/database

may show:

    password -> ..data/password
    username -> ..data/username

Plain `ls -l` therefore shows the permissions of the symbolic links rather than the actual target files.

Use:

    ls -lL /etc/database

The `-L` option tells `ls` to follow symbolic links.

This allows the actual permissions of the mounted Secret files to be inspected.

Another useful option is:

    stat -c '%a %n' /etc/database/password

### CKAD troubleshooting tip

When checking Secret volume permissions:

    ls -lL <mount-path>

rather than only:

    ls -l <mount-path>

---

## 15. Secret Updates: Environment Variables vs Volumes

Secret updates behave differently depending on how the Secret is consumed.

### Secret used as environment variable

If a running container receives:

    DB_PASSWORD

through `secretKeyRef` or `envFrom`, changing the Secret does NOT update the existing process environment.

The Pod/container must be recreated or restarted to receive the new environment value.

### Secret mounted as a volume

Mounted Secret files are updated automatically after a delay when the Secret changes.

However:

    Secret updated
          |
          v
    mounted file updated
          |
          v
    application uses new value?
          |
          v
    depends on whether the application rereads the file

The application may still need to be restarted if it only reads the file during startup.

---

## 16. Missing Required Secret Volume

By default, a referenced Secret is required:

    volumes:
      - name: secret-volume
        secret:
          secretName: missing-secret

If the Secret does not exist:

    volume setup fails
    Pod remains Pending

`kubectl describe pod` typically shows a `MountVolume.SetUp failed` event.

---

## 17. Optional Secret Volume

A Secret volume can be optional:

    volumes:
      - name: secret-volume
        secret:
          secretName: missing-secret
          optional: true

If the Secret does not exist:

    Pod can start
    mount path exists
    no Secret files are created

---

## 18. Missing Keys under `items`

Without `optional: true`:

    items:
      - key: API_TOKEN
        path: token

If `API_TOKEN` does not exist:

    volume setup fails
    Pod remains Pending

With:

    optional: true

the Pod can start.

Existing requested keys are mounted, while missing requested keys produce no file.

---

## 19. Missing Key with `secretKeyRef`

A required `secretKeyRef`:

    env:
      - name: API_TOKEN
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: API_TOKEN

If the key does not exist:

    Pod -> Pending
    Container -> Waiting
    Reason -> CreateContainerConfigError

A typical event is:

    couldn't find key API_TOKEN in Secret ...

---

## 20. Optional `secretKeyRef`

A specific Secret key reference can be optional:

    env:
      - name: API_TOKEN
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: API_TOKEN
            optional: true

If the key does not exist:

    Pod can start
    API_TOKEN is absent from the environment

Important:

    absent variable != variable with an empty value

For example:

    printenv API_TOKEN

returns no value and normally exits with code `1` if the variable is absent.

---

## 21. Missing Secret with `envFrom`

By default:

    envFrom:
      - secretRef:
          name: missing-secret

requires the Secret to exist.

If it does not exist:

    Pod -> Pending
    Container -> Waiting
    Reason -> CreateContainerConfigError

---

## 22. Optional Secret with `envFrom`

Use:

    envFrom:
      - secretRef:
          name: missing-secret
          optional: true

If the Secret does not exist:

    Pod can start
    no environment variables are imported from that Secret

---

## 23. Secret Types

### Generic credentials

Command:

    kubectl create secret generic ...

Type:

    Opaque

Typical use:

    usernames
    passwords
    API keys
    tokens

### TLS Secret

Command:

    kubectl create secret tls web-tls \
      --cert=tls.crt \
      --key=tls.key

Type:

    kubernetes.io/tls

Expected keys:

    tls.crt
    tls.key

### Docker Registry Secret

Command:

    kubectl create secret docker-registry registry-secret ...

Type:

    kubernetes.io/dockerconfigjson

Reference it from a Pod with:

    spec:
      imagePullSecrets:
        - name: registry-secret

`imagePullSecrets` is used to authenticate when pulling an image from a private registry.

It does NOT inject the registry credentials into the application container as environment variables.

---

## 24. Troubleshooting Secrets

Start with:

    kubectl get pod

Then:

    kubectl describe pod <pod-name>

Pay particular attention to:

    Status
    Containers
    Events

Typical Secret-related failures include:

    MountVolume.SetUp failed
    CreateContainerConfigError
    secret "<name>" not found
    couldn't find key <key> in Secret ...
    references non-existent secret key

If the container has not started, `kubectl logs` may not provide useful application logs.

A useful troubleshooting workflow is:

    kubectl get pod
          |
          v
    kubectl describe pod
          |
          v
    identify first blocking error
          |
          v
    fix and recreate
          |
          v
    inspect again
          |
          v
    identify next error
          |
          v
    verify with logs / exec

Troubleshooting multiple errors is often iterative because a later error may only become visible after an earlier blocking error is fixed.

---

## 25. Useful Verification Commands

List Secrets:

    kubectl get secrets

Inspect a Secret:

    kubectl get secret <secret-name> -o yaml

Decode one key:

    kubectl get secret <secret-name> \
      -o jsonpath='{.data.<KEY>}' | base64 -d; echo

Inspect Pod status:

    kubectl get pod <pod-name>

Inspect Pod configuration and events:

    kubectl describe pod <pod-name>

Check environment variables:

    kubectl exec <pod-name> -- env

Check one environment variable:

    kubectl exec <pod-name> -- printenv <VARIABLE>

List mounted Secret files:

    kubectl exec <pod-name> -- ls -l <mount-path>

Follow symbolic links to inspect actual file permissions:

    kubectl exec <pod-name> -- ls -lL <mount-path>

Read a mounted Secret file:

    kubectl exec <pod-name> -- cat <mount-path>/<file>

---

## CKAD Quick Reference

    # Generic Secret
    kubectl create secret generic app-secret \
      --from-literal=USER=admin \
      --from-literal=PASSWORD=secret123

    # Generate YAML
    kubectl create secret generic app-secret \
      --from-literal=USER=admin \
      --dry-run=client -o yaml

    # Secret from file
    kubectl create secret generic app-secret \
      --from-file=config.env

    # Secret from file with custom key
    kubectl create secret generic app-secret \
      --from-file=CONFIG=config.env

    # Secret from environment file
    kubectl create secret generic app-secret \
      --from-env-file=config.env

    # Inspect Secret
    kubectl get secret app-secret -o yaml

    # Decode one value
    kubectl get secret app-secret \
      -o jsonpath='{.data.PASSWORD}' | base64 -d; echo

    # Check mounted Secret permissions
    kubectl exec <pod> -- ls -lL <mount-path>

    # Troubleshoot
    kubectl get pod <pod>
    kubectl describe pod <pod>

---

## Key Exam Rules

1. Base64 is encoding, not encryption.
2. `data` expects Base64 values; `stringData` accepts plain-text input.
3. `envFrom.secretRef` imports all Secret keys as environment variables.
4. `secretKeyRef` selects an individual key and allows a different environment variable name.
5. Secret volume keys become files and their values become file contents.
6. `items` selects which keys are mounted and can rename them with `path`.
7. `defaultMode` controls default file permissions.
8. `item.mode` overrides `defaultMode` for an individual file.
9. `readOnly` controls the volume mount, not the Linux file mode.
10. Use `ls -lL` to follow Secret-volume symbolic links when checking actual permissions.
11. Secret-backed environment variables do not update in an already running container.
12. Mounted Secret files can update after the Secret changes.
13. Required missing Secrets or keys can prevent a container from starting or a volume from mounting.
14. `optional: true` allows the relevant missing Secret/key to be skipped.
15. `imagePullSecrets` is for private registry authentication, not application environment variables.
16. Use `kubectl describe pod` and Events early when troubleshooting Secret-related startup problems.