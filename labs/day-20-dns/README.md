# Day 20 — Kubernetes DNS

## 📌 Goal
Understand how DNS works inside Kubernetes and how Services are resolved to Pods.

---

## 🔹 Key Concepts

### 1. Why DNS is needed
Pods are ephemeral:
- Pod IPs change when Pods restart
- We cannot rely on Pod IPs

Solution:
- Use Services + DNS names

---

### 2. CoreDNS

CoreDNS is the DNS server inside Kubernetes.

Check:
    kubectl get pods -n kube-system

It resolves:
    service-name → ClusterIP

---

### 3. DNS Structure

Short name (same namespace):
    http://my-service

Full name (FQDN):
    http://my-service.default.svc.cluster.local

Structure:
    <service>.<namespace>.svc.cluster.local

---

### 4. Traffic Flow

Pod → DNS → ClusterIP → kube-proxy → Pod

Example:
    dns-service → 10.96.x.x → 10.244.x.x (Pod)

---

### 5. ClusterIP vs Pod IP

ClusterIP:
- Virtual IP
- Stable
- Represents Service
- Load balances traffic

Pod IP:
- Real IP
- Belongs to a single Pod
- Changes when Pod restarts

---

### 6. DNS Debugging

Test DNS:
    kubectl exec -it <pod> -- nslookup <service>

Test connectivity:
    kubectl exec -it <pod> -- wget -qO- <service>

---

## 🔹 Debug Strategy (CKAD Critical)

Step 1:
    nslookup <service>

If FAIL:
    → DNS/CoreDNS issue

If OK:
    → continue

Step 2:
    kubectl get svc

Step 3:
    kubectl get endpoints

If EMPTY:
    → selector / labels / readiness problem

If EXISTS:
    → port / app / container problem

Step 4:
    kubectl get pods --show-labels

Step 5:
    kubectl describe svc <service>

---

## 🔹 CoreDNS Debug

Check pods:
    kubectl get pods -n kube-system

Check logs:
    kubectl logs -n kube-system -l k8s-app=kube-dns

Check service:
    kubectl get svc -n kube-system kube-dns

Check endpoints:
    kubectl get endpoints -n kube-system kube-dns

---

## 🔹 Common Errors

1. Wrong Service selector
2. Pod labels mismatch
3. Pod not Ready
4. Wrong targetPort
5. No endpoints
6. CoreDNS not running

---

## 🔹 Key Learning

DNS working ≠ Service working

Always verify:
- DNS
- Endpoints
- Ports
- Pod health

---

## 🔹 Commands Summary

    kubectl get svc
    kubectl get endpoints
    kubectl get pods --show-labels
    kubectl describe svc <name>
    kubectl exec -it <pod> -- nslookup <service>
    kubectl exec -it <pod> -- wget -qO- <service>

---

## ✅ Result

You can:
- Resolve Services using DNS
- Understand ClusterIP vs Pod IP
- Debug DNS and Service issues (CKAD level)