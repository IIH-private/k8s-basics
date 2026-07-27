# Phase 3.4 – Namespaces

## Formål

Namespaces bruges til logisk opdeling af Kubernetes-ressourcer i et cluster. De gør det muligt at adskille workloads, teams og miljøer uden at oprette flere clustre.

---

## Standard namespaces

| Namespace | Formål |
|-----------|---------|
| `default` | Standard namespace for brugerressourcer. |
| `kube-system` | Kubernetes systemkomponenter. |
| `kube-public` | Offentligt tilgængelige cluster-informationer. |
| `kube-node-lease` | Bruges af kubelet til node heartbeats. |

---

## Cluster-scoped vs. Namespaced resources

### Cluster-scoped

- Node
- Namespace
- PersistentVolume
- ClusterRole
- ClusterRoleBinding

### Namespaced

- Pod
- Deployment
- ReplicaSet
- Service
- ConfigMap
- Secret
- Job
- CronJob

---

## Vigtige kommandoer

### Se namespaces

```bash
kubectl get namespaces
```

### Opret namespace

```bash
kubectl create namespace ckad-demo
```

### Opret namespace som YAML

```bash
kubectl create namespace ckad-demo \
  --dry-run=client \
  -o yaml > namespace.yaml
```

### Skift current namespace

```bash
kubectl config set-context --current --namespace=ckad-demo
```

### Se current namespace

```bash
kubectl config view --minify --output 'jsonpath={..namespace}{"\n"}'
```

### Arbejd i et bestemt namespace

```bash
kubectl get pods -n default
```

### Se ressourcer i alle namespaces

```bash
kubectl get pods -A
```

---

## Namespace i YAML

Et manifest kan angive namespace direkte:

```yaml
metadata:
  namespace: default
```

Hvis `metadata.namespace` mangler, bruges:

1. `-n <namespace>` hvis angivet.
2. Ellers current namespace fra contexten.

Hvis både `metadata.namespace` og `-n` bruges, skal de pege på samme namespace.

---

## CKAD-huskeregler

- Brug `-n` når kun enkelte kommandoer skal udføres i et andet namespace.
- Skift current namespace, hvis hele opgaven foregår i samme namespace.
- Brug `kubectl get pods -A`, hvis du ikke kender namespace.
- Brug labels ved verificering, når flere ressourcer findes i samme namespace.

Eksempel:

```bash
kubectl get pods -n default -l app=api
```