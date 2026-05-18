# Day 19 — Services Deep Dive

## 🎯 Goal
Forstå hvordan Kubernetes Services fungerer internt og lære at debugge dem korrekt (CKAD-niveau).

---

## 🧠 Key Concepts

### Service Flow
Service taler ikke direkte med Pods:

    Service → Endpoints → Pods

---

### Endpoints
Endpoints = liste af Pod IP’er, som matcher Service selector.

Tjek:

    kubectl get endpoints

Hvis tom:
- selector matcher ikke labels

---

### Ports

| Felt        | Betydning |
|-------------|----------|
| port        | Service port |
| targetPort  | Container port |
| nodePort    | Ekstern port |

---

### Readiness

    Running ≠ Ready

Pods skal være Ready for at modtage trafik.

---

## 🧪 Debugging Steps

1. Tjek Pods

    kubectl get pods

2. Tjek Service

    kubectl get svc
    kubectl describe svc <name>

3. Tjek Endpoints

    kubectl get endpoints

4. Tjek labels

    kubectl get pods --show-labels

5. Test fra cluster

    kubectl run test --rm -it --restart=Never --image=busybox:1.36 -- wget -qO- http://service-name

---

## ❌ Common Errors

- Selector mismatch
- Forkert targetPort
- Pods ikke Ready
- Forkerte labels
- Forkert namespace

---

## 🧪 Exam Task Fixes

### Problem 1 — Selector mismatch

Deployment:

    labels:
      app: api-v1

Selector:

    app: api

Fix:

    app: api

---

### Problem 2 — Readiness probe failure

Forkert:

    path: /health

Fix:

    path: /

---

## 🎯 Result

Test:

    kubectl run test --rm -it --restart=Never --image=busybox:1.36 -- wget -qO- http://api-service

Returnerer nginx HTML → Service virker.

---

## 🧠 Key Takeaway

Debug altid i denne rækkefølge:

    Pods → Labels → Endpoints → Ports → Readiness → Test

---