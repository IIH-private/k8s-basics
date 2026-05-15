# Day 17 — PersistentVolumeClaim (PVC) Deep Dive

## 🎯 Goal
Understand Kubernetes storage using:
- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)
- Dynamic provisioning (StorageClass)
- Static provisioning (manual PV)

---

## 🧠 Core Concepts

- **PV** = actual storage (disk)
- **PVC** = request for storage
- Pod never uses PV directly → always through PVC

---

## 🔁 PVC Lifecycle

1. Create PVC  
2. Kubernetes tries to match a PV  
3. If match → `Bound`  
4. If no match → `Pending`  
5. Pod uses PVC → gets storage  

---

## ⚙️ Dynamic Provisioning (StorageClass)

Flow:
PVC → StorageClass → PV (auto-created)

Key behavior:
- PVC may be `Pending`
- Reason: `WaitForFirstConsumer`
- PV created only when Pod uses PVC

Check:
kubectl get pvc  
kubectl get pv  
kubectl get storageclass  

---

## 📦 Static Provisioning

Flow:
Create PV → PVC binds immediately

Requirements (must match):
- storage size
- accessModes
- storageClassName

---

## 🔐 Access Modes

- ReadWriteOnce (RWO) → one node
- ReadOnlyMany (ROX) → many pods (read-only)
- ReadWriteMany (RWX) → many pods (read/write)

Note:
RWO allows multiple Pods on the same node

---

## 🔄 Reclaim Policy

Defines what happens when PVC is deleted:

- Delete → PV and storage deleted
- Retain → PV and data remain
- Recycle → deprecated

Test:
kubectl delete pvc <name>  
kubectl get pv  

---

## 🧪 Dynamic Provisioning Example

### PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

### Pod
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo hello >> /data/file.txt && sleep 3600"]
      volumeMounts:
        - name: storage
          mountPath: /data
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: demo-pvc

Verify:
kubectl exec -it demo-pod -- cat /data/file.txt  
kubectl delete pod demo-pod  
kubectl apply -f pod.yaml  
kubectl exec -it demo-pod -- cat /data/file.txt  

Expected:
hello  
hello  

---

## 🧪 Static Provisioning Example

### PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  hostPath:
    path: /mnt/data

### PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi

### Pod
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "echo STATIC >> /data/file.txt && sleep 3600"]
      volumeMounts:
        - name: storage
          mountPath: /data
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: manual-pvc

---

## 🐞 Debugging Patterns (CKAD)

PVC `Pending`:
- no matching PV
- storageClass mismatch
- accessMode mismatch
- size mismatch

Pod cannot mount:
kubectl describe pod <name>

Common issue:
- wrong claimName

---

## 🧠 Key Takeaways

- PVC = persistent storage for Pods
- Dynamic provisioning = default
- Static provisioning = manual control
- Reclaim Policy controls lifecycle
- Debugging is critical

---

## 🚀 CKAD Mindset

1. Check PVC status  
2. If `Pending` → check matching  
3. If Pod fails → use `describe`  
4. Think: match → bind → use  

---

## 📌 Summary

Pod → PVC → PV  

Delete order:  
Pod → PVC → PV  

---