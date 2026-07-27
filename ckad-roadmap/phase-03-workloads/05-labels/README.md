# Phase 3.5 – Labels

## Formål

Labels er **key/value-par**, som bruges til at beskrive Kubernetes-objekter. De bruges af Kubernetes til at gruppere, finde og administrere ressourcer via **selectors**.

---

## Grundlæggende

Et label består af en **key** og en **value**.

Eksempel:

```yaml
metadata:
  labels:
    app: nginx
    env: prod
    tier: frontend
```

Labels ligger altid under:

```yaml
metadata:
  labels:
```

---

## Se labels

Vis labels for alle Pods:

```bash
kubectl get pods --show-labels
```

Vis detaljer om en Pod:

```bash
kubectl describe pod <pod-navn>
```

Vis Pod som YAML:

```bash
kubectl get pod <pod-navn> -o yaml
```

---

## Label selectors

Find Pods med en bestemt label:

```bash
kubectl get pods -l app=nginx
```

De vigtigste selectors:

```bash
kubectl get pods -l app=nginx
kubectl get pods -l app!=nginx
kubectl get pods -l app
kubectl get pods -l '!app'
kubectl get pods -l 'app in (nginx,tools)'
kubectl get pods -l 'app notin (nginx)'
```

---

## Tilføje, ændre og fjerne labels

Tilføj et label:

```bash
kubectl label pod <pod-navn> version=v1
```

Overskriv et eksisterende label:

```bash
kubectl label pod <pod-navn> env=prod --overwrite
```

Fjern et label:

```bash
kubectl label pod <pod-navn> version-
```

---

## Labels og selectors

- **Labels** beskriver et objekt.
- **Selectors** bruges til at finde objekter med bestemte labels.

Eksempel:

```yaml
labels:
  app: nginx
```

Selector:

```text
app=nginx
```

Selectoren finder alle objekter med labelen `app=nginx`.

---

## Sammenhæng med ReplicaSets og Deployments

ReplicaSets og Deployments finder de Pods, de skal administrere, ved hjælp af **selectors**.

Derfor skal Deploymentens selector matche labels på Pod-template'en.

Eksempel:

```yaml
selector:
  matchLabels:
    app: nginx

template:
  metadata:
    labels:
      app: nginx
```

Hvis selector og labels ikke matcher, kan Deploymenten ikke oprettes.

---

## Husk

- Labels er **metadata**.
- Flere objekter kan have samme label.
- Selectors søger efter labels – ikke objektnavne.
- `--overwrite` bruges til at ændre et eksisterende label.
- Et label fjernes med et efterstillet `-`.

---

## CKAD-kommandoer

```bash
kubectl get pods --show-labels

kubectl get pods -l app=nginx

kubectl get pods -l app

kubectl get pods -l '!app'

kubectl get pods -l 'app in (nginx,tools)'

kubectl label pod <pod-navn> version=v1

kubectl label pod <pod-navn> env=prod --overwrite

kubectl label pod <pod-navn> version-
```