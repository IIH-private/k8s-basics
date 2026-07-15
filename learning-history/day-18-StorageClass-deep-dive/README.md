# Day 18 — StorageClass Deep Dive (CKAD)

## Goal
Understand StorageClass, dynamic provisioning, and how to debug PVC issues in CKAD.

---

## Core Concept

StorageClass is used for **dynamic provisioning**.

Mental model:

PVC → triggers → StorageClass → creates → PV → Pod uses it

---

## Static vs Dynamic

Static:
PV (manual) → PVC → Pod

Dynamic:
PVC → StorageClass → PV (auto) → Pod

---

## Key Rule

StorageClass does NOT create anything by itself.

PVC triggers creation.

---

## Default StorageClass

Check:

kubectl get sc

If PVC does NOT define storageClassName → default StorageClass is used.

---

## Common Debug Case

PVC stuck in Pending:

kubectl get pvc
kubectl describe pvc <name>

Check:

kubectl get sc

---

## Typical Errors

1. StorageClass does not exist
2. Wrong storageClassName
3. No provisioner
4. WaitForFirstConsumer
5. Pod not using PVC
6. Wrong claimName in Pod

---

## Immutable Field (IMPORTANT)

storageClassName cannot be changed after PVC is created.

Wrong:

kubectl edit pvc <name>

Correct:

kubectl delete pvc <name>
kubectl apply -f pvc.yaml

---

## WaitForFirstConsumer

If StorageClass uses:

volumeBindingMode: WaitForFirstConsumer

PVC will stay Pending until a Pod uses it.

Event example:

waiting for first consumer to be created before binding

---

## Working Example

PVC:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

Pod:

apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: test-pvc

Apply:

kubectl apply -f pvc.yaml
kubectl apply -f pod.yaml

---

## Debug Example (Real Task)

Problems:
- Missing StorageClass
- Wrong claimName

Fix:
- Change storageClassName to existing (e.g. standard)
- Fix claimName
- Delete + recreate PVC

---

## CKAD Debug Flow

1. kubectl get pvc
2. kubectl describe pvc
3. kubectl get sc
4. Check StorageClass exists
5. Check provisioner
6. Check volumeBindingMode
7. Create Pod if needed
8. Check claimName
9. Delete + recreate PVC if needed
10. Verify

---

## Key Takeaways

- PVC triggers PV creation
- StorageClass defines how
- Default StorageClass is used automatically
- storageClassName is immutable
- WaitForFirstConsumer delays binding
- Always use kubectl describe pvc

---

## Useful Commands

kubectl get sc
kubectl get pvc
kubectl describe pvc <name>
kubectl get pv
kubectl get pod
kubectl describe pod <name>
kubectl apply -f <file>
kubectl delete pvc <name>
kubectl delete pod <name>

---

## Cleanup

kubectl delete pod test-pod
kubectl delete pvc test-pvc