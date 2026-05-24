# Day 22 — Kubernetes ServiceAccounts

## 📌 Overview
Today I learned how Kubernetes ServiceAccounts work and how Pods use them to interact with the Kubernetes API.

---

## 🧠 Core Concepts

### ServiceAccount
- A ServiceAccount is an identity used by Pods
- Namespace-scoped
- Every Pod uses one (default if not specified)

Check existing ServiceAccounts:
    kubectl get sa

---

### Default Behavior
- Pods use `default` ServiceAccount if none specified
- Kubernetes automatically mounts a token into the Pod

Path inside container:
    /var/run/secrets/kubernetes.io/serviceaccount/

---

### Token (Credential)
- Used for authentication to Kubernetes API
- Automatically mounted unless disabled
- Contains:
  - token
  - ca.crt
  - namespace

---

### ServiceAccount vs Token vs RBAC

| Component        | Purpose        |
|----------------|---------------|
| ServiceAccount | Identity       |
| Token          | Credential     |
| RBAC           | Permissions    |

---

## 🔐 Disable Auto-Mount (Security)

Disable token injection:

    automountServiceAccountToken: false

### Effect:
- No token inside Pod
- No API authentication possible

### Why:
- Reduces attack surface
- Follows least privilege principle

---

## 📦 Create ServiceAccount

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: app-sa

Apply:
    kubectl apply -f sa.yaml

---

## 🚀 Use in Pod

    apiVersion: v1
    kind: Pod
    metadata:
      name: sa-pod
    spec:
      serviceAccountName: app-sa
      containers:
      - name: app
        image: nginx

---

## 🚀 Use in Deployment (IMPORTANT)

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: sa-deploy
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: test
      template:
        metadata:
          labels:
            app: test
        spec:
          serviceAccountName: app-sa
          automountServiceAccountToken: false
          containers:
          - name: app
            image: nginx

---

## 🔍 Debugging

### Pod not created

    kubectl describe rs <replicaset>

Typical error:
    serviceaccount "xxx" not found

---

### API access test

    kubectl exec -it <pod> -- sh

    curl https://kubernetes.default

Possible failures:
- Token missing
- automount disabled
- missing RBAC (403 Forbidden)

---

## 🧪 Verify token inside Pod

    ls /var/run/secrets/kubernetes.io/serviceaccount/

- Exists → token mounted
- Missing → automount disabled

---

## ⚠️ CKAD Key Points

- ServiceAccount = identity only
- Token = authentication
- RBAC = authorization
- automountServiceAccountToken controls token presence
- Pod setting overrides ServiceAccount
- No token = no API access

---

## 🎯 Summary

ServiceAccounts define who the Pod is, but not what it can do.
RBAC is required for permissions.