# Day 16 — Kubernetes Persistent Volumes (PV & PVC)

## 🎯 Goal

Understand how Kubernetes provides persistent storage using:

* PersistentVolume (PV)
* PersistentVolumeClaim (PVC)
* Pod volume mounts

---

## 🧠 Core Concept

```
Pod → PVC → PV → Physical Storage
```

* **Pod** uses storage
* **PVC** requests storage
* **PV** provides storage

---

## 🔑 Key Principles

* PV is a **cluster resource**
* PVC is a **request for storage**
* Kubernetes automatically **binds PVC → PV**
* Data is stored in PV, not in the Pod
* Data survives Pod deletion

---

## ⚠️ emptyDir vs PV

| Feature       | emptyDir  | PV/PVC          |
| ------------- | --------- | --------------- |
| Lifetime      | Pod       | Independent     |
| Data persists | ❌ No      | ✅ Yes           |
| Use case      | Temp data | Persistent data |

---

## 🧪 Step 1 — Create PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-day16
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/day16-data
```

```bash
kubectl apply -f pv.yaml
kubectl get pv
```

Expected:

```
STATUS: Available
```

---

## 🧪 Step 2 — Create PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-day16
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
```

Expected:

```
STATUS: Bound
```

---

## 🔗 Binding Behavior

* PV must match:

  * storage size
  * accessModes
  * storageClassName

If not:

```
PVC → Pending
```

---

## 🧪 Step 3 — Create Pod using PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-day16
spec:
  containers:
    - name: app
      image: busybox
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: storage
          mountPath: /data
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: pvc-day16
```

```bash
kubectl apply -f pod.yaml
kubectl get pod
```

---

## 🧪 Step 4 — Write data manually

```bash
kubectl exec -it pod-day16 -- sh
```

Inside container:

```sh
echo REAL-DATA > /data/test.txt
exit
```

Verify:

```bash
kubectl exec -it pod-day16 -- cat /data/test.txt
```

---

## 🧪 Step 5 — Persistence Test

```bash
kubectl delete pod pod-day16
kubectl apply -f pod.yaml
kubectl exec -it pod-day16 -- cat /data/test.txt
```

Expected:

```
REAL-DATA
```

---

## 🧠 What this proves

* Pod is ephemeral ❌
* Data is persistent ✅
* Storage lives in PV

---

## ⚠️ Common Errors (CKAD)

### 1. StorageClass mismatch

```
PVC uses default (standard)
PV uses manual
→ No binding
```

Fix:

```yaml
storageClassName: manual
```

---

### 2. AccessMode mismatch

```
PVC: ReadWriteMany
PV:  ReadWriteOnce
→ Pending
```

---

### 3. Capacity mismatch

```
PVC requests more than PV provides
→ Pending
```

---

### 4. Wrong claimName

```
persistentvolumeclaim "xxx" not found
```

Fix:

```yaml
claimName: correct-name
```

---

## 🧠 Important Rules

```
1 PV = 1 PVC
```

* PV is not shared by default
* Unused space is not reusable

---

## 🔁 Lifecycle

```
Pod deleted ❌
PVC exists ✅
PV exists ✅
Data exists ✅
```

---

## 🔥 CKAD Debug Mindset

When something fails:

1. `kubectl get pv,pvc,pod`
2. `kubectl describe pvc`
3. `kubectl describe pod`
4. Look for:

   * Pending
   * Not found
   * Mismatch

---

## 🎯 Summary

* PV = storage
* PVC = request
* Pod = consumer
* Data persists outside Pod
* Binding must match exactly
* Debugging is about identifying mismatches

```
```
