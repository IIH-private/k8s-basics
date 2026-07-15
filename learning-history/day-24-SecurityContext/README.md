# Day 24 — Kubernetes SecurityContext Deep Dive

## Goal
Learn how Kubernetes SecurityContext improves container security and how to debug common CKAD-style failures.

---

# What was learned

## 1. Containers run as root by default

Test:

    kubectl exec -it sec-test -- sh
    id

Result:

    uid=0(root) gid=0(root)

Meaning:
- container runs as root by default
- potentially dangerous

---

# runAsNonRoot

    securityContext:
      runAsNonRoot: true

Purpose:
- Kubernetes validates that container does NOT run as root

Important:
- does NOT change user
- only checks

Common CKAD error:

    container has runAsNonRoot and image will run as root

---

# runAsUser

    securityContext:
      runAsUser: 1000

Purpose:
- explicitly sets runtime user

Verification:

    kubectl exec -it sec-test -- sh
    id

Result:

    uid=1000 gid=0(root)

---

# Difference between runAsNonRoot and runAsUser

| Setting | Purpose |
|---|---|
| runAsNonRoot | validation/check |
| runAsUser | sets runtime user |

Best practice:

    securityContext:
      runAsNonRoot: true
      runAsUser: 1000

---

# readOnlyRootFilesystem

    securityContext:
      readOnlyRootFilesystem: true

Purpose:
- prevents writes to root filesystem

Important CKAD debugging pattern:

    Read-only file system

Usually means:
- application needs writable directories

---

# Real nginx debugging

nginx failed because it needed writable access to:

    /var/cache/nginx
    /run

Error examples:

    mkdir() "/var/cache/nginx/client_temp" failed (30: Read-only file system)

    open() "/run/nginx.pid" failed (30: Read-only file system)

---

# Fix with writable volumes

    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /run

    volumes:
    - name: cache
      emptyDir: {}
    - name: run
      emptyDir: {}

Result:
- nginx starts successfully
- root filesystem still protected

---

# Capabilities

Example:

    capabilities:
      drop:
        - ALL

Purpose:
- removes Linux capabilities from container

Verification:

    cat /proc/1/status | grep Cap

Result:

    CapInh: 0000000000000000
    CapPrm: 0000000000000000
    CapEff: 0000000000000000

Meaning:
- container has zero capabilities

Important:
- container can still work if app does not require capabilities

---

# containerPort clarification

    ports:
    - containerPort: 80

Important:
- mostly metadata/documentation
- does NOT open port
- does NOT grant privileges
- does NOT configure networking

Real binding happens inside application itself.

---

# Important CKAD Debugging Patterns

## Pattern 1 — runAsNonRoot failure

Error:

    container has runAsNonRoot and image will run as root

Cause:
- image tries running as root

Fix:
- add non-root user
- or use compatible image

---

## Pattern 2 — Read-only filesystem

Error:

    Read-only file system

Cause:
- application tries writing to protected filesystem

Fix:
- mount writable volume

---

## Pattern 3 — Capability restrictions

Possible symptoms:
- permission denied
- bind failures
- networking limitations

Fix:
- add only required capabilities

---

# Final secure nginx example

    apiVersion: v1
    kind: Pod
    metadata:
      name: secure-nginx
    spec:
      containers:
      - name: app
        image: nginx
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          readOnlyRootFilesystem: true
          capabilities:
            drop:
              - ALL
        volumeMounts:
        - name: cache
          mountPath: /var/cache/nginx
        - name: run
          mountPath: /run
      volumes:
      - name: cache
        emptyDir: {}
      - name: run
        emptyDir: {}

---

# Key CKAD Takeaways

When you see:

    readOnlyRootFilesystem: true

Always think:

    What paths does the application need to write to?

Typical writable paths:
- /tmp
- /run
- /var/cache
- /var/log

---

# Useful Commands

    kubectl describe pod <pod>
    kubectl logs <pod>
    kubectl exec -it <pod> -- sh
    id
    cat /proc/1/status | grep Cap