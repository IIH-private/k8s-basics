\# Day 03 – Kubernetes Pods \& Debugging



\## 🎯 Goal



Learn how to debug failing Pods and understand what happens at runtime vs YAML configuration.



\---



\## 🔧 Topics covered



\* kubectl describe

\* kubectl logs

\* kubectl logs --previous

\* kubectl exec

\* ImagePullBackOff

\* CrashLoopBackOff

\* Runtime debugging inside container



\---



\## 🧪 Scenarios



\### 1. Broken image



\* Used invalid image: nginx:doesnotexist

\* Result: ImagePullBackOff

\* Learned: Kubernetes cannot pull image → Pod never starts



\---



\### 2. CrashLoopBackOff



\* Container exits with error

\* Kubernetes restarts it repeatedly

\* Backoff delay increases over time



\---



\### 3. Port mismatch



\* containerPort set to 1234

\* nginx actually listens on port 80

\* Pod still Running



\---



\## 🧠 Key learnings



\* `kubectl get pods` shows status, not root cause

\* `kubectl describe` shows events and errors

\* `kubectl logs` shows application output

\* `kubectl logs --previous` shows logs from last crash

\* `kubectl exec` allows runtime inspection



\---



\## ⚠️ Important concepts



\### containerPort is NOT enforced



Kubernetes does not verify that the application listens on the declared port.



\---



\### App working vs App accessible



\* App working = process is running

\* App accessible = traffic can reach it (Service required)



\---



\## 🔍 Debug approach



1\. kubectl get pods

2\. kubectl describe pod

3\. kubectl logs

4\. kubectl logs --previous (if restart)

5\. kubectl exec (inspect container)



\---



\## 📁 Files



\* broken.yaml

\* crash.yaml

\* wrong-port.yaml



\---



\## 🚀 Summary



Debugging in Kubernetes is about:



\* understanding events

\* checking logs

\* verifying runtime behavior



YAML alone is not enough.



