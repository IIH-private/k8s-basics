# Day 23 — RBAC (Role + RoleBinding)

## Goal
Understand how Kubernetes controls access using RBAC:
- Role (permissions)
- RoleBinding (who gets permissions)

---

## Key Concepts

### Role
Defines WHAT actions are allowed on WHICH resources.

Example:
- get pods
- list pods

---

### RoleBinding
Connects:
- ServiceAccount (WHO)
- Role (WHAT)

---

## Basic Flow

ServiceAccount → RoleBinding → Role

---

## Example

### Role

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: pod-reader
      namespace: default
    rules:
    - apiGroups:
      - ""
      resources:
      - pods
      verbs:
      - get
      - list

---

### RoleBinding

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: read-pods-binding
      namespace: default
    subjects:
    - kind: ServiceAccount
      name: app-sa
      namespace: default
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io

---

## Testing Permissions

Check access:

    kubectl auth can-i list pods \
      --as=system:serviceaccount:default:app-sa \
      -n default

Expected result:

    yes

---

## API Access from Pod

Inside pod:

    TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
    CA=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

    curl --cacert $CA \
      -H "Authorization: Bearer $TOKEN" \
      https://kubernetes.default.svc/api/v1/namespaces/default/pods

---

## Important Lessons

- RBAC controls authorization (not authentication or TLS)
- ServiceAccount provides identity
- Role defines permissions
- RoleBinding assigns permissions

---

## Common Mistakes

- Missing verb (get vs list)
- Wrong namespace
- Wrong apiGroup
- Pod not using correct ServiceAccount
- Using cluster-wide API with namespace Role

---

## CKAD Tips

- Always use minimal permissions
- Always verify with kubectl auth can-i
- Remember:
  
      get ≠ list