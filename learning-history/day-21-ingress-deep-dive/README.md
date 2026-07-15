# Day 21 — Ingress (L7 routing, traffic flow, rewrite rules)

## 🎯 Goal

Understand and practice:

* L7 routing (Ingress)
* Real traffic flow
* Path-based routing
* Host-based routing
* Rewrite rules
* Debugging Ingress issues (CKAD-style)

---

## 🧠 Core Concepts

### Ingress vs Service

Service (L4):

* Works on IP + port
* Does NOT understand HTTP

Ingress (L7):

* Understands HTTP
* Can route based on:

  * Host (app1.local)
  * Path (/app1)

---

## 🔁 Traffic Flow

Client → Ingress Controller → Ingress → Service → Pod

---

## 🔧 Path-based routing

Example:

```
/app1 → app1-service
/app2 → app2-service
```

### Problem

nginx only understands:

```
/
```

Request:

```
/app1 → 404
```

### Solution (rewrite)

```
nginx.ingress.kubernetes.io/rewrite-target: /
```

Effect:

```
/app1 → / → nginx → OK
```

---

## 🌐 Host-based routing

Example:

```
app1.local → app1-service
app2.local → app2-service
```

### Important

Test using Host header:

```
curl -H "Host: app1.local" http://localhost:8080
```

---

## 🔑 Key Rules

* Host match happens BEFORE path match
* Rewrite happens AFTER routing
* Service uses labels → NOT deployments
* containerPort is NOT used for routing
* targetPort MUST match real application port

---

## 🧪 Hands-on (summary)

### Create apps

```
k create deployment host-app1 --image=nginx:1.25 --dry-run=client -o yaml > app1.yaml
k create service clusterip host-app1-service --tcp=80:80 --dry-run=client -o yaml >> app1.yaml

k create deployment host-app2 --image=nginx:1.25 --dry-run=client -o yaml > app2.yaml
k create service clusterip host-app2-service --tcp=80:80 --dry-run=client -o yaml >> app2.yaml
```

### Ingress (manual)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: app1.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: host-app1-service
                port:
                  number: 80
    - host: app2.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: host-app2-service
                port:
                  number: 80
```

---

## 🔌 Access (kind)

Use port-forward:

```
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80
```

Test:

```
curl -H "Host: app1.local" http://localhost:8080
curl -H "Host: app2.local" http://localhost:8080
```

---

## 🐞 Debug patterns (CKAD critical)

### 503 → No endpoints

```
k get endpoints
```

Cause:

* Wrong selector

---

### 502 → Backend unreachable

Cause:

* Wrong targetPort

---

### 404 → Path/host mismatch

Cause:

* Wrong path
* Missing rewrite
* Wrong host

---

### Wrong app returned

Cause:

* Ingress points to wrong Service

---

## 🧠 Final Mental Model

Ingress:

* Decides WHERE (Service)

Service:

* Decides WHICH Pods

Rewrite:

* Decides WHAT path Pod sees

---

## 🚀 Key Takeaways

* Host-based routing is cleaner (no rewrite)
* Path-based routing often requires rewrite
* Debug layer-by-layer:

  1. Pod
  2. Service
  3. Ingress
* Always verify:

  * selectors
  * ports
  * paths
  * host

---

## ✅ Status

✔ Path-based routing understood
✔ Host-based routing implemented
✔ Rewrite rules mastered
✔ Full Ingress debugging flow completed
